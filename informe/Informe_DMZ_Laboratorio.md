# Informe de configuración de DMZ con Cisco Packet Tracer

### 1. Objetivo del laboratorio

Configurar una Zona Desmilitarizada (DMZ) usando un router Cisco ISR 2911 (`Router_FW`), aplicando NAT estático y Listas de Control de Acceso (ACLs) para permitir que un servidor web alojado en la DMZ sea accesible desde una red externa por HTTP, sin comprometer la seguridad de la red interna (LAN). El objetivo final es que la DMZ aísle servicios críticos, controle el tráfico entre zonas y exponga de forma controlada el servicio web, manteniendo la red interna protegida de accesos no autorizados.

---

### 2. Topología implementada

- **Cantidad de redes:** 3 (LAN interna, DMZ, Red externa), todas en /24 sobre direccionamiento privado 192.168.x.0/24.
- **Dispositivos usados:**
  - 1 Router Cisco ISR 2911 (`Router_FW`), actuando como firewall perimetral con 3 interfaces Gigabit Ethernet.
  - 3 Switches Cisco 2960-24TT (`SW_Internal`, `SW_DMZ`, `SW_External`).
  - 1 PC en la red interna (`PC_Internal`).
  - 1 Servidor web en la DMZ (`Server-PT Web_DMZ`).
  - 1 PC en la red externa, simulando un usuario de Internet (`PC_External`).

- **Función de cada zona:**
  - **LAN (INTERNAL NETWORK – Gi0/0):** Red interna de confianza. No debe ser alcanzable desde la DMZ ni desde el exterior, pero sí puede iniciar conexiones hacia ambas.
  - **DMZ (Gi0/1):** Zona intermedia donde se alojan servicios públicos (el servidor web). Puede recibir tráfico desde el exterior y responder a la LAN, pero no puede iniciar conexiones hacia la LAN.
  - **Red Externa (EXTERNAL NETWORK – Gi0/2):** Simula Internet. Solo tiene permitido acceder al servicio HTTP publicado del servidor de la DMZ; no puede hacer ping al router ni acceder a la LAN.

---

### 3. Plan de direccionamiento IP

| Dispositivo             | IP              | Máscara           | Gateway           |
|--------------------------|-----------------|--------------------|--------------------|
| PC_Internal              | 192.168.1.10    | 255.255.255.0      | 192.168.1.1        |
| Server_DMZ (Web_DMZ)     | 192.168.2.10    | 255.255.255.0      | 192.168.2.1        |
| PC_External              | 192.168.3.10    | 255.255.255.0      | 192.168.3.1        |
| Router_FW Gi0/0 (LAN)    | 192.168.1.1     | 255.255.255.0      | —                  |
| Router_FW Gi0/1 (DMZ)    | 192.168.2.1     | 255.255.255.0      | —                  |
| Router_FW Gi0/2 (Ext)    | 192.168.3.1     | 255.255.255.0      | —                  |

---

### 4. Configuración aplicada (resumen)

**Interfaces del router:**

```bash
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 ip nat inside
 ip access-group 101 in
 no shutdown

interface GigabitEthernet0/2
 ip address 192.168.3.1 255.255.255.0
 ip nat outside
 ip access-group 102 in
 no shutdown
```

**NAT estático (publica el puerto 80 del servidor DMZ en la IP externa del router):**

```bash
ip nat inside source static tcp 192.168.2.10 80 192.168.3.1 80
```

**ACL 102 — aplicada en Gi0/2 (tráfico entrante desde la red externa):**

```bash
access-list 102 permit tcp any host 192.168.3.1 eq 80
access-list 102 deny icmp any host 192.168.3.1
access-list 102 deny ip any 192.168.1.0 0.0.0.255
access-list 102 permit ip any any
```

**ACL 101 — aplicada en Gi0/1 (tráfico entrante desde la DMZ):**

```bash
access-list 101 permit icmp 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255 echo-reply
access-list 101 permit tcp 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255 established
access-list 101 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
access-list 101 permit ip any any
```

> **Nota técnica:** la ACL 101 necesitó dos líneas de excepción (`echo-reply` y `established`) para permitir que el tráfico de **respuesta** generado por la DMZ (ante un ping o una conexión HTTP iniciados desde la LAN) pudiera regresar a la LAN, sin abrir la puerta a que la DMZ **inicie** conexiones nuevas hacia la red interna. Esta es la diferencia clave entre "bloquear toda comunicación" y "bloquear solo las conexiones iniciadas por la DMZ".

---

### 5. Verificaciones realizadas

| # | Prueba | Origen → Destino | Resultado |
|---|--------|-------------------|-----------|
| 1 | Ping | PC_Internal → 192.168.1.1 (gateway LAN) | ✅ Responde |
| 2 | Ping | Web_DMZ → 192.168.2.1 (gateway DMZ) | ✅ Responde |
| 3 | Ping | PC_External → 192.168.3.1 (router, red externa) | ✅ Bloqueado (Destination host unreachable) |
| 4 | TCP/HTTP | PC_Internal → 192.168.2.10 (Web_DMZ) | ✅ Carga la página web |
| 5 | Ping | Web_DMZ → 192.168.1.10 (PC_Internal) | ✅ Bloqueado (Destination host unreachable) |
| 6 | HTTP | PC_External → http://192.168.3.1 (vía NAT) | ✅ Carga la página del servidor DMZ |

Resultado final del validador de Packet Tracer: **"Congratulations Guest! You completed the activity."**

---

### 6. Conclusiones y recomendaciones

Este laboratorio permitió aplicar de forma práctica los tres pilares de una DMZ: segmentación de red, NAT y control de acceso mediante ACLs. El reto principal no fue la configuración inicial de direcciones IP o NAT, sino ajustar las ACLs para que el tráfico de **retorno** (respuestas a conexiones legítimamente iniciadas desde la LAN) no quedara bloqueado por error junto con el tráfico que sí se quería impedir (conexiones iniciadas por la DMZ hacia la LAN). Esto se resolvió usando las palabras clave `echo-reply` (para ICMP) y `established` (para TCP), que permiten distinguir el tráfico de respuesta del tráfico de inicio de conexión.

Como recomendación, antes de aplicar cualquier ACL es importante verificar la conectividad básica entre zonas (ping a los gateways), ya que un error en el direccionamiento puede hacer parecer que una ACL está mal configurada cuando en realidad el problema es de IP o de ruteo. También se recomienda probar tanto el tráfico de ida como el de vuelta al validar una regla de seguridad, ya que las ACLs estándar de Cisco no son "con estado" por defecto y pueden bloquear respuestas legítimas si no se contempla explícitamente.

---

### 7. Capturas de evidencia

Ver carpeta `evidencias/` del repositorio, que incluye:

- Configuración de IPs en el router (CLI) y en los tres hosts finales.
- Aplicación de NAT y ACLs en el router.
- Resultado de los 6 ping/HTTP de validación (capturas 1 a 6 de la tabla de la sección 5).
- Captura final de `show running-config` confirmando la configuración persistida.
- Captura del mensaje "Congratulations Guest! You completed the activity." del validador de Packet Tracer.

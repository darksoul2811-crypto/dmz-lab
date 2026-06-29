# dmz-lab

Laboratorio de implementación de una **Zona Desmilitarizada (DMZ)** en Cisco Packet Tracer, configurando direccionamiento IP, NAT estático y ACLs para exponer de forma controlada un servidor web manteniendo protegida la red interna (LAN).

## Objetivo

Construir y asegurar una red con tres zonas (LAN, DMZ y Red Externa) a través de un router Cisco ISR 2911 que actúa como firewall perimetral, de modo que:

- El servidor web de la DMZ sea accesible por HTTP desde Internet (red externa), vía NAT estático.
- La red externa no pueda hacer ping al router ni acceder a la LAN.
- La DMZ no pueda iniciar conexiones hacia la LAN, aunque sí pueda responder a conexiones iniciadas por la LAN.

## Contenido del repositorio

```
dmz-lab/
├── README.md
├── DMZ_PROJECT.pka              # Archivo final de Packet Tracer
├── informe/
│   └── Informe_DMZ_Laboratorio.md   # Informe técnico (plantilla completada)
└── evidencias/
    └── ...                       # Capturas de las pruebas realizadas
```

## Resultado

El laboratorio fue validado por el sistema de evaluación de Packet Tracer ("Check Results"), confirmando el cumplimiento de las 6 pruebas de conectividad definidas en la actividad (ver detalle en `informe/Informe_DMZ_Laboratorio.md`).

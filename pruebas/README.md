# Evidencias de pruebas

Las pruebas de conectividad valen **30 pts**. Guardar aquí capturas (`.png`) de cada prueba exitosa.

Nombre sugerido: `faseN-origen-destino-prueba.png`, p. ej. `fase1-GRE-VENTAS-a-GRE-PRODUCCION-ping.png`.

| Carpeta | Qué va |
| --- | --- |
| `grecia/`, `san-ramon/`, `perez-zeledon/` | Ping entre VLANs de la sede, DHCP (`ipconfig`), SSH a cada equipo, `show vlan brief`, `show ip interface brief` |
| `wan/` | Ping entre sedes, `show ip route`, prueba de respaldo (cortar un enlace y verificar que sigue habiendo conectividad) |

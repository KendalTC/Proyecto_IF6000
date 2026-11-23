# Evidencias de pruebas

Las pruebas de conectividad valen **30 pts** (+10 opcionales por la ruta de respaldo).
Todas se tomaron sobre `pkt/ClavitoVeloz.pkt` el **22/11/2026**. Cada imagen trae una franja con el número de evidencia, qué se probó y el resultado.

Cómo repetir las pruebas: [comandos-verificacion.md](comandos-verificacion.md).

## Índice

| # | Archivo | Qué demuestra | Requisito del enunciado |
| --- | --- | --- | --- |
| 01 | [grecia/01-GRE-ssh-GRE-SW01-banner.png](grecia/01-GRE-ssh-GRE-SW01-banner.png) | SSH a GRE-SW01 desde la laptop admin, con banner | Punto 7: SSH · Punto 4: banner |
| 02 | [grecia/02-GRE-ssh-RT01-SW02-SW03-3-usuarios.png](grecia/02-GRE-ssh-RT01-SW02-SW03-3-usuarios.png) | SSH a GRE-RT01, SW02 y SW03 con los 3 administradores (kendal, mariangel, sebas) | Punto 7: 3 usuarios |
| 03 | [grecia/03-GRE-telnet-rechazado.png](grecia/03-GRE-telnet-rechazado.png) | Telnet rechazado: el único acceso remoto es SSH | Punto 7: único acceso SSH |
| 04 | [grecia/04-GRE-dhcp-ventas-y-ping-produccion.png](grecia/04-GRE-dhcp-ventas-y-ping-produccion.png) | GRE-VEN-PC01 recibe IP por DHCP (.86.7 /27) y hace ping a Producción | Punto 5: DHCP |
| 05 | [grecia/05-GRE-ping-entre-vlans.png](grecia/05-GRE-ping-entre-vlans.png) | Ping de Ventas a Desarrollo, Gerencia y laptop admin | Punto 2: VLANs interconectadas |
| 06 | [grecia/06-GRE-gerencia-ip-estatica.png](grecia/06-GRE-gerencia-ip-estatica.png) | Gerencia Grecia con IP estática .98 /28 (menos de 10 PCs) | Punto 5: estáticas |
| 07 | [san-ramon/07-SR-ssh-4-equipos.png](san-ramon/07-SR-ssh-4-equipos.png) | SSH a SR-RT01 y SR-SW01–03 | Punto 7 |
| 08 | [san-ramon/08-SR-dhcp-bodega-y-ping-ventas.png](san-ramon/08-SR-dhcp-bodega-y-ping-ventas.png) | SR-BOD-PC01 recibe IP por DHCP (.86.142 /27) y hace ping a Ventas | Punto 8: DHCP obligatorio |
| 09 | [san-ramon/09-SR-ping-entre-vlans.png](san-ramon/09-SR-ping-entre-vlans.png) | Ping de Bodega a Gerencia, Contabilidad y laptop admin | Punto 8 |
| 10 | [perez-zeledon/10-PZ-ssh-4-equipos.png](perez-zeledon/10-PZ-ssh-4-equipos.png) | SSH a PZ-RT01 y PZ-SW01–03 | Punto 7 |
| 11 | [perez-zeledon/11-PZ-dhcp-bodega-y-ping-ventas.png](perez-zeledon/11-PZ-dhcp-bodega-y-ping-ventas.png) | PZ-BOD-PC01 recibe IP por DHCP (.87.15 /27) y hace ping a Ventas | Punto 9: DHCP obligatorio |
| 12 | [perez-zeledon/12-PZ-ping-entre-vlans.png](perez-zeledon/12-PZ-ping-entre-vlans.png) | Ping de Bodega a Gerencia, Contabilidad y laptop admin | Punto 9 |
| 13 | [wan/13-WAN-ssh-entre-sedes.png](wan/13-WAN-ssh-entre-sedes.png) | SSH desde Grecia a los routers de SR y PZ | Punto 7: administración remota |
| 14 | [wan/14-WAN-ping-grecia-a-SR-y-PZ.png](wan/14-WAN-ping-grecia-a-SR-y-PZ.png) | Ping Grecia → San Ramón y Pérez Zeledón | Punto 10: conectividad total |
| 15 | [wan/15-WAN-ping-SR-a-grecia-y-PZ.png](wan/15-WAN-ping-SR-a-grecia-y-PZ.png) | Ping San Ramón → Grecia y Pérez Zeledón | Punto 10 |
| 16 | [wan/16-WAN-ping-PZ-a-grecia-y-SR.png](wan/16-WAN-ping-PZ-a-grecia-y-SR.png) | Ping Pérez Zeledón → Grecia y San Ramón | Punto 10 |
| 17 | [wan/17-WAN-respaldo-1-ruta-normal.png](wan/17-WAN-respaldo-1-ruta-normal.png) | `tracert` PZ → Grecia por el enlace directo (.87.138) | Punto 11 (opcional +10) |
| 18 | [wan/18-WAN-respaldo-2-enlace-caido.png](wan/18-WAN-respaldo-2-enlace-caido.png) | Con GRE-RT01 Gi0/2 apagada, EIGRP desvía por San Ramón (.87.133 → .87.129) | Punto 11 |
| 19 | [wan/19-WAN-respaldo-3-restaurado.png](wan/19-WAN-respaldo-3-restaurado.png) | Enlace restaurado: vuelve a la ruta directa | Punto 11 |

## Verificación de configuración (CLI)

Además de las capturas, el 22/11 se revisó la configuración real de los 12 equipos (`show ip interface brief`, `show ip eigrp neighbors`, `show ip route`, `show ip dhcp binding`, `show interfaces trunk`, `show interfaces status`, `show running-config`):

| Revisión | Resultado |
| --- | --- |
| Interfaces de los 3 routers | Todas **up/up** (Gi0/0, subinterfaces, Gi0/1, Gi0/2) |
| EIGRP AS 10 | **2 vecinos** en cada router; 10 rutas **D** a las sedes remotas |
| DHCP | Grecia 22/20/16 · SR 20/16/12/10 · PZ 20/16/12/10 IPs entregadas (una por PC) |
| Trunks | Native VLAN **100** en todos; sin mismatch |
| Puertos sin uso | En **disabled** según D4 |
| SSH | Version 2.0 en los 12 equipos (PZ-RT01 corregido de 1.99 a 2.0) |

Respaldo de la configuración real: `configs/<sede>/<EQUIPO>/<EQUIPO>-running-config.txt`.

## Leer los resultados

- **"Request timed out" en el primer paquete:** normal; es la resolución ARP.
- **TTL=127:** el paquete cruzó 1 router (misma sede, otra VLAN). **TTL=126:** cruzó 2 routers (otra sede).
- **Telnet "Open" y luego "closed by foreign host":** el router acepta la conexión TCP pero la corta porque las líneas VTY solo permiten SSH.

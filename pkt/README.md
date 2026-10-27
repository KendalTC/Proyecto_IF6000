# Archivos Packet Tracer

## Convención de nombres

| Archivo | Ubicación | Descripción |
| --- | --- | --- |
| `ClavitoVeloz.pkt` | `pkt/` | Versión de trabajo actual (la que se edita) |
| `faseN-descripcion.pkt` | `pkt/versiones/` | Respaldo al cerrar cada fase, p. ej. `fase1-grecia-vlans-dhcp.pkt` |
| `NOMBRE-NOMBRE-NOMBRE.pkt` | `pkt/` | Archivo final de entrega (al cerrar la Fase 6) |

## Reglas

1. **Una sola persona edita a la vez.** Avisar en el grupo antes de abrir el .pkt y hacer `git pull`.
2. Al terminar: guardar, `git add`, `git commit`, `git push`, y avisar que quedó libre.
3. Al cerrar una fase, copiar el .pkt a `versiones/` con su nombre de fase (respaldo exigido por el enunciado).
4. Versión de Packet Tracer del grupo: _(anotar aquí, p. ej. 8.2.2)_ — todos deben usar la misma.

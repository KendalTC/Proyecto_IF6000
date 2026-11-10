# Configuraciones por equipo

Una carpeta por equipo. Los `.txt` se crean sobre la marcha, al configurar cada fase, con el nombre `<EQUIPO>-faseN.txt` (por ejemplo `GRE-RT01-fase1.txt`). Los archivos de una fase se aplican **encima** de los de la fase anterior.

> **No escribir dentro de `.gitkeep`.** Ese archivo solo existe para que Git suba la carpeta vacía. Siempre crear un `.txt` nuevo (en GitHub: **Add file → Create new file**, escribiendo el nombre con la ruta, p. ej. `configs/grecia/GRE-RT01/GRE-RT01-fase2.txt`).

| Sede | Router | Switches |
| --- | --- | --- |
| `grecia/` | `GRE-RT01/` | `GRE-SW01/`, `GRE-SW02/`, `GRE-SW03/` |
| `san-ramon/` | `SR-RT01/` | `SR-SW01/`, `SR-SW02/`, `SR-SW03/` |
| `perez-zeledon/` | `PZ-RT01/` | `PZ-SW01/`, `PZ-SW02/`, `PZ-SW03/` |

## Encabezado acordado para cada .txt

Todo comentario empieza con `!`, así el archivo completo se puede pegar en la CLI sin quitar nada.

```
!============================================================
! PROYECTO IF6000 - Ferreteria El Clavito Veloz
! Sede ......: 
! Equipo ....: 
! Bloque ....: 
! Fase ......: 
! Que hace ..: 
! Requiere ..: (archivos que deben aplicarse antes)
! Decisiones : (D1.x, D2.x … del README principal)
! Autor .....: (nombre y fecha)
! Estado ....: [ ] Pendiente  [ ] Aplicado  [ ] Probado
!------------------------------------------------------------
! VERIFICAR : comando -> resultado esperado
! SI FALLA  : error típico -> causa
!============================================================
```

Reglas:

- Comentar cada grupo de comandos con el porqué (sirve como guía de estudio).
- Si cambia una IP, puerto o VLAN, se cambia primero en el README principal.
- Al cerrar cada fase, guardar el `show running-config` real del equipo en su carpeta.

## Estado

| Equipo | Fase 1 | Fase 2 | Fase 3 | Fase 5 (seguridad) |
| --- | --- | --- | --- | --- |
| GRE-RT01 | ✅ | ✅ WAN a SR + EIGRP | ✅ WAN a PZ | ✅ |
| GRE-SW01 | ✅ | — | — | ✅ |
| GRE-SW02 | ✅ | — | — | ✅ |
| GRE-SW03 | ✅ | — | — | ✅ |
| SR-RT01 | — | ✅ | ✅ WAN a PZ | ✅ |
| SR-SW01 · SR-SW02 · SR-SW03 | — | ✅ | — | ✅ |
| PZ-RT01 | — | — | ✅ | ✅ |
| PZ-SW01 · PZ-SW02 · PZ-SW03 | — | — | ✅ | ✅ |

Leyenda: ⬜ sin aplicar · 🟨 aplicado · ✅ probado. La Fase 4 (PCs) no tiene scripts: se hizo en Packet Tracer.

Los `-fase5.txt` tienen el mismo formato que las fases 1–3 (se pueden pegar completos en la CLI). Revalidación final de SSH: [`docs/checklist-entrega.md`](../docs/checklist-entrega.md).

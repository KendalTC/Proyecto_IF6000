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

| Equipo | Archivos | Estado |
| --- | --- | --- |
| GRE-RT01 | `GRE-RT01-fase1.txt` · `GRE-RT01-fase2.txt` | ✅ Fase 1 probada · ⬜ Fase 2 (WAN + EIGRP) sin aplicar |
| GRE-SW01 | `GRE-SW01-fase1.txt` | ✅ Probado (07/11) |
| GRE-SW02 | `GRE-SW02-fase1.txt` | ✅ Probado (07/11) |
| GRE-SW03 | `GRE-SW03-fase1.txt` | ✅ Probado (07/11) |
| SR-RT01 | `SR-RT01-fase2.txt` | ⬜ Listo, sin aplicar |
| SR-SW01 | `SR-SW01-fase2.txt` | ⬜ Listo, sin aplicar |
| SR-SW02 | `SR-SW02-fase2.txt` | ⬜ Listo, sin aplicar |
| SR-SW03 | `SR-SW03-fase2.txt` | ⬜ Listo, sin aplicar |
| PZ-RT01 · PZ-SW01 · PZ-SW02 · PZ-SW03 | — | ⬜ Sin configurar |

Leyenda: ⬜ sin aplicar · 🟨 aplicado · ✅ probado (con capturas en `pruebas/`).

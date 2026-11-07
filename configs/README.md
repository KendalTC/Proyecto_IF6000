# Configuraciones por equipo

Una carpeta por equipo. Los `.txt` se crean sobre la marcha, al configurar cada fase.

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
! Requiere ..: (bloques que deben aplicarse antes)
! Decisiones : (D1.x, D2.x … del README principal)
! Estado ....: [ ] Pendiente  [ ] Aplicado  [ ] Probado
!------------------------------------------------------------
! VERIFICAR : comando -> resultado esperado
! SI FALLA  : error típico -> causa
! DESHACER  : comandos para revertir
!============================================================
```

Reglas:

- Comentar cada grupo de comandos con el porqué (sirve como guía de estudio).
- Si cambia una IP, puerto o VLAN, se cambia primero en el README principal.
- Al cerrar cada fase, guardar el `show running-config` real del equipo en su carpeta.

## Estado

| Equipo | Estado |
| --- | --- |
| GRE-RT01 · GRE-SW01 · GRE-SW02 · GRE-SW03 | ⬜ Sin configurar |
| SR-RT01 · SR-SW01 · SR-SW02 · SR-SW03 | ⬜ Sin configurar |
| PZ-RT01 · PZ-SW01 · PZ-SW02 · PZ-SW03 | ⬜ Sin configurar |

# Proyecto IF6000 – Ferretería "El Clavito Veloz"

Red corporativa en **Cisco Packet Tracer** para 3 sedes: Grecia (Zona Franca), San Ramón y Pérez Zeledón.
Curso IF6000 Redes en los Negocios II – UCR, II Ciclo 2026. Entrega presencial: **jueves 26 de noviembre de 2026**.

Este README es la **fuente de verdad del diseño**: toda decisión tomada (redes, VLANs, puertos, IPs) se registra aquí antes de configurarla en Packet Tracer.

## Estado actual (09/11/2026)

La red está completa: 3 sedes, 15 VLANs, DHCP, triángulo WAN con EIGRP, los PCs y SSH en los 12 equipos. Falta el cierre: evidencias, archivo final y revisión con el [checklist de entrega](docs/checklist-entrega.md). **Entrega: jueves 26/11/2026.**

| Fase | Estado |
| --- | --- |
| 0 – Decisiones + topología física | ✅ 26/10 |
| 1 – Grecia LAN | ✅ 07/11 (configuró Sebastián el 03/11) |
| 2 – San Ramón + WAN GRE–SR + EIGRP | ✅ 07/11 |
| 3 – Pérez Zeledón + triángulo WAN | ✅ 07/11 (falta confirmar la prueba de respaldo) |
| 4 – Poblar los PCs | ✅ 09/11 (Jose) |
| 5 – Seguridad (SSH, usuarios, banner, puertos) | ✅ 08/11 (Jose) — revalidar con el [checklist de entrega](docs/checklist-entrega.md) |
| 6 – Pruebas finales + archivo de entrega + guía de estudio | ⬜ |

**Próxima sesión:** ver [Pendientes](#pendientes).

---

## Contenido

1. [Estructura del repositorio](#estructura-del-repositorio)
2. [Cómo trabajamos (Git)](#cómo-trabajamos-git)
3. [Integrantes](#integrantes) · [Resumen del proyecto](#resumen-del-proyecto)
4. [Plan por fases](#plan-por-fases)
5. [Bitácora de decisiones](#bitácora-de-decisiones)
6. [D1 – Direccionamiento (VLSM)](#d1--direccionamiento-vlsm)
7. [D2 – Equipos y VLANs](#d2--equipos-y-vlans)
8. [D3 – Puertos, conexiones e IPs fijas](#d3--puertos-conexiones-e-ips-fijas)
9. [D4 – Seguridad (Fase 5)](#d4--seguridad-fase-5)
10. [Fase 4 – PCs](#fase-4--pcs)
11. [Topología física (Fase 0)](#topología-física-fase-0)
12. [Problemas encontrados y solución](#problemas-encontrados-y-solución)
13. [Pendientes](#pendientes)
14. [Comandos de referencia (vistos en clase)](#comandos-de-referencia-vistos-en-clase)

---

## Estructura del repositorio

```
1.Proyecto/
├── README.md                  ← Diseño y decisiones (este archivo)
├── docs/
│   ├── enunciado/             ← PDF oficial del proyecto
│   ├── checklist-entrega.md   ← Revisión final antes de entregar
│   └── apuntes-clase/         ← Laboratorios y comandos vistos en clase
├── pkt/
│   ├── README.md              ← Convención de nombres de los .pkt
│   └── versiones/             ← Respaldo de cada fase (fase1, fase2, …)
├── configs/                   ← Scripts de configuración por equipo (.txt)
│   ├── grecia/
│   ├── san-ramon/
│   └── perez-zeledon/
└── pruebas/                   ← Evidencias: capturas de ping, SSH, show …
    ├── grecia/
    ├── san-ramon/
    ├── perez-zeledon/
    └── wan/
```

## Cómo trabajamos (Git)

- **Una sola persona edita el .pkt a la vez.** Packet Tracer guarda en binario: Git no puede mezclar dos versiones del mismo .pkt. Antes de abrirlo, avisar en el chat del grupo y hacer `git pull`.
- Al terminar: guardar, copiar el respaldo a `pkt/versiones/` y hacer commit + push.
- Los cambios de diseño se hacen primero en este README (sección Bitácora) y después en Packet Tracer.
- Mensajes de commit sugeridos: `fase1: VLANs y DHCP Grecia`, `docs: decisión D4 usuarios SSH`.

```bash
git pull                     # antes de empezar
git add .
git commit -m "fase1: router-on-a-stick Grecia"
git push                     # al terminar
```

---

## Integrantes

| Integrante | Aportes principales |
| --- | --- |
| Kendall (KendalTC) | Diseño y decisiones (D1–D3), topología física, Fases 2 y 3, repositorio |
| Sebastián Alfaro | Fase 1 – Grecia |
| Jose Solorzano | Fase 4 – PCs y Fase 5 – Seguridad |
| Mariangel | Capturas de pruebas (pendiente) |
| Alejandro | _(por definir)_ |

Usuarios SSH: el enunciado exige **exactamente 3 administradores** en todos los equipos, por eso hay 3 usuarios (`kendal`, `mariangel`, `sebas`) aunque el grupo sea de 5.

## Resumen del proyecto

| Sede | Departamento | PCs | Asignación IP |
| --- | --- | --- | --- |
| Grecia (Zona Franca) | Gerencia | 8 | Estática (< 10 equipos) |
| Grecia (Zona Franca) | Producción | 20 | DHCP |
| Grecia (Zona Franca) | Ventas | 22 | DHCP |
| Grecia (Zona Franca) | Desarrollo (QA & Cybersecurity) | 16 | DHCP |
| San Ramón | Gerencia / Ventas / Bodega / Contabilidad | 12 / 16 / 20 / 10 | DHCP (obligatorio) |
| Pérez Zeledón | Gerencia / Ventas / Bodega / Contabilidad | 12 / 16 / 20 / 10 | DHCP (obligatorio) |

**Total: 182 PCs** (66 Grecia + 58 por sucursal).

Requisitos clave del enunciado:

- VLAN por departamento, router-on-a-stick (subinterfaces 802.1Q), un segmento como VLAN nativa.
- Subneteo VLSM propio del grupo, sin desperdicio.
- Hardening: hostname, descripciones, banner, contraseñas.
- VLAN administrativa por sede; acceso **solo por SSH** con **exactamente 3 usuarios administradores** en todos los equipos.
- DHCP con exclusiones; conectividad total entre las 3 sedes.
- Opcional (+10 pts): 2 o más rutas de respaldo con conmutación automática.
- Rúbrica: configuración 50 · pruebas de conectividad 30 · presentación/cableado/subneteo 15 · comprobación 5.
- Archivo final: `NOMBRE-NOMBRE-NOMBRE.PKT`.

---

## Plan por fases

Cada fase termina con un .pkt funcional guardado en `pkt/versiones/`.

| Fase | Contenido | Estado |
| --- | --- | --- |
| 0 | Decisiones base + topología física sin configurar (3 sedes, triángulo WAN) | ✅ Listo (26/10) |
| 1 | Grecia LAN: hostname y descripciones; VLANs, trunks, subinterfaces, DHCP, Gerencia estática, VLAN 900 con IPs; pings | ✅ Listo (07/11) |
| 2 | San Ramón LAN (misma receta) + enlace WAN GRE–SR + enrutamiento | ✅ Listo (07/11) |
| 3 | Pérez Zeledón LAN + triángulo WAN + prueba de respaldo (opcional) | ✅ Listo (07/11) |
| 4 | Poblar con los 182 PCs; etiquetado y presentación | ✅ Listo (09/11, Jose) |
| 5 | **Seguridad** en los 12 equipos: enable/consola, 3 usuarios, SSH, banner, `shutdown` de puertos sin uso | ✅ Listo (08/11, Jose) |
| 6 | Batería final de pruebas ([checklist](docs/checklist-entrega.md)) + guía para la comprobación | ⏭️ Siguiente |

> La seguridad va al final (D3.4) para construir sin contraseñas. La VLAN 900 y sus IPs sí se configuran desde la Fase 1 porque son parte de la estructura.

---

## Bitácora de decisiones

Ninguna red, máscara o IP se fija sin consultarla con el grupo. Más reciente primero.

| # | Fecha | Decisión | Motivo |
| --- | --- | --- | --- |
| D4.3 | 2026-11-08 | Puertos sin uso con `description SIN-USO-PENDIENTE-SHUTDOWN-FASE5` + `shutdown`; las interfaces de los routers están todas en uso | Hardening del enunciado (punto 4) |
| D4.2 | 2026-11-08 | Consola y VTY con `login local` y `exec-timeout 5 0`; VTY 0–15 solo `transport input ssh`; SSH v2 con RSA 2048; dominio `clavitoveloz.com` | "Único acceso mediante SSH" (punto 7) |
| D4.1 | 2026-11-08 | 3 administradores en los 12 equipos: `kendal`, `mariangel`, `sebas` (privilege 15, `secret`); `enable secret`; `service password-encryption`; banner MOTD | "Exactamente 3 usuarios administradores en todos los dispositivos" (punto 7) |
| D5.1 | 2026-11-07 | Enrutamiento entre sedes: **EIGRP AS 10**, `network 172.27.86.0 0.0.1.255` + `no auto-summary` en los 3 routers (propuesta en uso, falta confirmación del grupo) | Visto en clase (semanas 10–11); una sola línea cubre la /23; el triángulo da el respaldo automático |
| D3.4 | 2026-10-26 | Seguridad (contraseñas, usuarios, SSH, banner, `shutdown`) en la Fase 5, al final; VLAN 900 e IPs desde la Fase 1 | Construir y probar sin autenticarse en cada equipo |
| D3.3 | 2026-10-26 | Fase 1 arranca con router + 3 switches cableados y solo 1–2 PCs por VLAN | Estructura definitiva desde el inicio; los PCs se agregan en Fase 5 |
| D3.2 | 2026-10-26 | IPs fijas en VLAN admin (router, SW01–03, laptop); Gerencia Grecia .98–.105; DHCP excluye solo el gateway | Orden fijo y fácil de recordar |
| D3.1 | 2026-10-26 | Switches en cadena (RT → SW01 → SW02 → SW03); un departamento principal por switch; triángulo WAN Gi0/1 → Gi0/2 | Igual al laboratorio de troncales; etiquetado limpio |
| D2.5 | 2026-10-26 | Nombres SEDE-TIPO## (GRE-RT01, GRE-SW01…03, SR-…, PZ-…); dominio `clavitoveloz.com` | Identifica sede y función a simple vista |
| D2.4 | 2026-10-26 | Puertos sin uso: solo `shutdown` | Igual que en clase |
| D2.3 | 2026-10-26 | IDs de VLAN en centenas, mismo ID por departamento en todas las sedes; nativa = 100 GERENCIA | Estilo de clase; la nativa es un segmento (punto 6) |
| D2.2 | 2026-10-26 | Switch Cisco 2960-24TT, 3 por sede | 3 switches dan 73 puertos libres (24N+1); 2 no alcanzan |
| D2.1 | 2026-10-26 | Router Cisco 2911 en las 3 sedes | 3 puertos Gigabit: 1 LAN + 2 WAN, sin módulos |
| D1.6 | 2026-10-26 | Gateway = primera IP usable de cada subred | Convención estándar, la de clase |
| D1.5 | 2026-10-26 | Red base **172.27.86.0/23** (172.27.86.0 – 172.27.87.255) | Privada, alineada a /23, poco común |
| D1.4 | 2026-10-26 | Un bloque /25 por sede; último /25 para WAN y reserva | Orden, sumarización, crecimiento aislado |
| D1.3 | 2026-10-26 | Contabilidad /28 y Admin /29 | Ya cumplen +20%; subirlas sería desperdicio |
| D1.2 | 2026-10-26 | Política de crecimiento: +20% sobre los PCs actuales | Cabe en /23; Gerencia sucursal pasa a /27 |
| D1.1 | 2026-10-26 | Hosts = PCs + gateway; Admin = 5 (gateway + 3 SW + laptop); WAN = 3 enlaces /30 | Sin impresoras ni servidores (no los pide el enunciado) |

---

## D1 – Direccionamiento (VLSM)

Red base **172.27.86.0/23**: 372 de 512 direcciones asignadas. Cálculo verificado con script.

### Bloques por sede

| Bloque | Rango | Uso | Reserva libre |
| --- | --- | --- | --- |
| 172.27.86.0/25 | .86.0 – .86.127 | Grecia | .86.120 – .86.127 (8) |
| 172.27.86.128/25 | .86.128 – .86.255 | San Ramón | .86.248 – .86.255 (8) |
| 172.27.87.0/25 | .87.0 – .87.127 | Pérez Zeledón | .87.120 – .87.127 (8) |
| 172.27.87.128/25 | .87.128 – .87.255 | Enlaces WAN + reserva | .87.140 – .87.255 (116) |

### Tabla VLSM

| Sede | Subred | Red | Máscara | Rango usable | Broadcast | Gateway | Usables | Usadas | Libres |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Grecia | Ventas | 172.27.86.0 | /27 (255.255.255.224) | .86.1 – .86.30 | .86.31 | .86.1 | 30 | 23 | 7 |
| Grecia | Producción | 172.27.86.32 | /27 (255.255.255.224) | .86.33 – .86.62 | .86.63 | .86.33 | 30 | 21 | 9 |
| Grecia | Desarrollo | 172.27.86.64 | /27 (255.255.255.224) | .86.65 – .86.94 | .86.95 | .86.65 | 30 | 17 | 13 |
| Grecia | Gerencia | 172.27.86.96 | /28 (255.255.255.240) | .86.97 – .86.110 | .86.111 | .86.97 | 14 | 9 | 5 |
| Grecia | Admin | 172.27.86.112 | /29 (255.255.255.248) | .86.113 – .86.118 | .86.119 | .86.113 | 6 | 5 | 1 |
| San Ramón | Bodega | 172.27.86.128 | /27 (255.255.255.224) | .86.129 – .86.158 | .86.159 | .86.129 | 30 | 21 | 9 |
| San Ramón | Ventas | 172.27.86.160 | /27 (255.255.255.224) | .86.161 – .86.190 | .86.191 | .86.161 | 30 | 17 | 13 |
| San Ramón | Gerencia | 172.27.86.192 | /27 (255.255.255.224) | .86.193 – .86.222 | .86.223 | .86.193 | 30 | 13 | 17 |
| San Ramón | Contabilidad | 172.27.86.224 | /28 (255.255.255.240) | .86.225 – .86.238 | .86.239 | .86.225 | 14 | 11 | 3 |
| San Ramón | Admin | 172.27.86.240 | /29 (255.255.255.248) | .86.241 – .86.246 | .86.247 | .86.241 | 6 | 5 | 1 |
| Pérez Zeledón | Bodega | 172.27.87.0 | /27 (255.255.255.224) | .87.1 – .87.30 | .87.31 | .87.1 | 30 | 21 | 9 |
| Pérez Zeledón | Ventas | 172.27.87.32 | /27 (255.255.255.224) | .87.33 – .87.62 | .87.63 | .87.33 | 30 | 17 | 13 |
| Pérez Zeledón | Gerencia | 172.27.87.64 | /27 (255.255.255.224) | .87.65 – .87.94 | .87.95 | .87.65 | 30 | 13 | 17 |
| Pérez Zeledón | Contabilidad | 172.27.87.96 | /28 (255.255.255.240) | .87.97 – .87.110 | .87.111 | .87.97 | 14 | 11 | 3 |
| Pérez Zeledón | Admin | 172.27.87.112 | /29 (255.255.255.248) | .87.113 – .87.118 | .87.119 | .87.113 | 6 | 5 | 1 |

### Enlaces WAN (/30 – 255.255.255.252)

| Enlace | Red | Extremo A | Extremo B | Broadcast |
| --- | --- | --- | --- | --- |
| Grecia – San Ramón | 172.27.87.128 | .87.129 (GRE Gi0/1) | .87.130 (SR Gi0/2) | .87.131 |
| San Ramón – Pérez Zeledón | 172.27.87.132 | .87.133 (SR Gi0/1) | .87.134 (PZ Gi0/2) | .87.135 |
| Pérez Zeledón – Grecia | 172.27.87.136 | .87.137 (PZ Gi0/1) | .87.138 (GRE Gi0/2) | .87.139 |

---

## D2 – Equipos y VLANs

### Inventario y compras

| Sede | Equipo | Modelo | Necesarios | Existentes | Por comprar |
| --- | --- | --- | --- | --- | --- |
| Grecia | Router | 2911 | 1 | 1 | 0 |
| Grecia | Switch | 2960-24TT | 3 | 2 | 1 |
| Grecia | Computadoras (66 PCs + 1 laptop admin) | PC / Laptop | 67 | 26 | 41 |
| San Ramón | Router | 2911 | 1 | 0 | 1 |
| San Ramón | Switch | 2960-24TT | 3 | 0 | 3 |
| San Ramón | Computadoras (58 PCs + 1 laptop admin) | PC / Laptop | 59 | 0 | 59 |
| Pérez Zeledón | Router | 2911 | 1 | 0 | 1 |
| Pérez Zeledón | Switch | 2960-24TT | 3 | 0 | 3 |
| Pérez Zeledón | Computadoras (58 PCs + 1 laptop admin) | PC / Laptop | 59 | 0 | 59 |
| **Total** | | | **188** | **29** | **2 routers · 7 switches · 159 computadoras** |

Switches: con N switches 2960 en cadena quedan **24N + 1** puertos para equipos finales. N = 2 → 49 (no alcanza); N = 3 → 73.

Cableado: **directo** PC–switch y switch–router · **cruzado** switch–switch y router–router.

### VLANs (mismo ID por departamento en todas las sedes)

| VLAN | Nombre | Sedes | Nota |
| --- | --- | --- | --- |
| 100 | GERENCIA | Todas | **Nativa** en todos los trunks (ambos extremos) |
| 200 | PRODUCCION | Grecia | |
| 300 | VENTAS | Todas | |
| 400 | DESARROLLO | Grecia | |
| 500 | BODEGA | San Ramón, Pérez Zeledón | |
| 600 | CONTABILIDAD | San Ramón, Pérez Zeledón | |
| 900 | ADMIN-SSH | Todas | Gestión por SSH |

### VLAN → subred → subinterfaz

| VLAN | Sede | Subred | Gateway (subinterfaz) | Asignación |
| --- | --- | --- | --- | --- |
| 100 | Grecia | 172.27.86.96/28 | 172.27.86.97 (gi0/0.100, `native`) | Estática |
| 200 | Grecia | 172.27.86.32/27 | 172.27.86.33 (gi0/0.200) | DHCP |
| 300 | Grecia | 172.27.86.0/27 | 172.27.86.1 (gi0/0.300) | DHCP |
| 400 | Grecia | 172.27.86.64/27 | 172.27.86.65 (gi0/0.400) | DHCP |
| 900 | Grecia | 172.27.86.112/29 | 172.27.86.113 (gi0/0.900) | Estática |
| 100 | San Ramón | 172.27.86.192/27 | 172.27.86.193 (gi0/0.100, `native`) | DHCP |
| 300 | San Ramón | 172.27.86.160/27 | 172.27.86.161 (gi0/0.300) | DHCP |
| 500 | San Ramón | 172.27.86.128/27 | 172.27.86.129 (gi0/0.500) | DHCP |
| 600 | San Ramón | 172.27.86.224/28 | 172.27.86.225 (gi0/0.600) | DHCP |
| 900 | San Ramón | 172.27.86.240/29 | 172.27.86.241 (gi0/0.900) | Estática |
| 100 | Pérez Zeledón | 172.27.87.64/27 | 172.27.87.65 (gi0/0.100, `native`) | DHCP |
| 300 | Pérez Zeledón | 172.27.87.32/27 | 172.27.87.33 (gi0/0.300) | DHCP |
| 500 | Pérez Zeledón | 172.27.87.0/27 | 172.27.87.1 (gi0/0.500) | DHCP |
| 600 | Pérez Zeledón | 172.27.87.96/28 | 172.27.87.97 (gi0/0.600) | DHCP |
| 900 | Pérez Zeledón | 172.27.87.112/29 | 172.27.87.113 (gi0/0.900) | Estática |

### Nombres de equipos

| Sede | Router | Switches |
| --- | --- | --- |
| Grecia | GRE-RT01 | GRE-SW01, GRE-SW02, GRE-SW03 |
| San Ramón | SR-RT01 | SR-SW01, SR-SW02, SR-SW03 |
| Pérez Zeledón | PZ-RT01 | PZ-SW01, PZ-SW02, PZ-SW03 |

Dominio (SSH): `clavitoveloz.com` · Puertos sin uso: `shutdown`.

---

## D3 – Puertos, conexiones e IPs fijas

### Enlaces troncales y WAN

| Origen | Puerto | Destino | Puerto | Tipo | Cable |
| --- | --- | --- | --- | --- | --- |
| XX-RT01 | Gi0/0 | XX-SW01 | Gi0/1 | Trunk (subinterfaces) | Directo |
| XX-SW01 | Gi0/2 | XX-SW02 | Gi0/1 | Trunk, nativa 100 | Cruzado |
| XX-SW02 | Gi0/2 | XX-SW03 | Gi0/1 | Trunk, nativa 100 | Cruzado |
| GRE-RT01 | Gi0/1 | SR-RT01 | Gi0/2 | WAN 172.27.87.128/30 | Cruzado |
| SR-RT01 | Gi0/1 | PZ-RT01 | Gi0/2 | WAN 172.27.87.132/30 | Cruzado |
| PZ-RT01 | Gi0/1 | GRE-RT01 | Gi0/2 | WAN 172.27.87.136/30 | Cruzado |

XX = GRE, SR o PZ (igual en las 3 sedes).

### Puertos de acceso

| Sede | Switch | Puertos | VLAN | Uso |
| --- | --- | --- | --- | --- |
| Grecia | GRE-SW01 | Fa0/1–22 | 300 | Ventas (22) |
| Grecia | GRE-SW01 | Fa0/24 | 900 | Laptop admin |
| Grecia | GRE-SW02 | Fa0/1–20 | 200 | Producción (20) |
| Grecia | GRE-SW03 | Fa0/1–16 | 400 | Desarrollo (16) |
| Grecia | GRE-SW03 | Fa0/17–24 | 100 | Gerencia (8) |
| Sucursales | XX-SW01 | Fa0/1–20 | 500 | Bodega (20) |
| Sucursales | XX-SW01 | Fa0/24 | 900 | Laptop admin |
| Sucursales | XX-SW02 | Fa0/1–16 | 300 | Ventas (16) |
| Sucursales | XX-SW03 | Fa0/1–12 | 100 | Gerencia (12) |
| Sucursales | XX-SW03 | Fa0/13–22 | 600 | Contabilidad (10) |

Todo puerto no listado: `shutdown`.

### IPs fijas – VLAN 900 ADMIN-SSH

| Equipo | Grecia (.86.112/29) | San Ramón (.86.240/29) | Pérez Zeledón (.87.112/29) |
| --- | --- | --- | --- |
| Router gi0/0.900 | 172.27.86.113 | 172.27.86.241 | 172.27.87.113 |
| SW01 (interface vlan 900) | 172.27.86.114 | 172.27.86.242 | 172.27.87.114 |
| SW02 (interface vlan 900) | 172.27.86.115 | 172.27.86.243 | 172.27.87.115 |
| SW03 (interface vlan 900) | 172.27.86.116 | 172.27.86.244 | 172.27.87.116 |
| Laptop admin | 172.27.86.117 | 172.27.86.245 | 172.27.87.117 |
| Libre | 172.27.86.118 | 172.27.86.246 | 172.27.87.118 |

Los switches usan `ip default-gateway` = IP del router en su VLAN 900.

### Gerencia Grecia (estática – 172.27.86.96/28)

| PC | IP | Máscara | Gateway |
| --- | --- | --- | --- |
| GRE-GER-PC01 … PC08 | 172.27.86.98 – 172.27.86.105 | 255.255.255.240 | 172.27.86.97 |

Libres para crecer: .106 – .110.

### Pools DHCP (en el router de cada sede)

| Pool | Red | default-router | excluded-address |
| --- | --- | --- | --- |
| GRE-VENTAS | 172.27.86.0 255.255.255.224 | 172.27.86.1 | 172.27.86.1 |
| GRE-PRODUCCION | 172.27.86.32 255.255.255.224 | 172.27.86.33 | 172.27.86.33 |
| GRE-DESARROLLO | 172.27.86.64 255.255.255.224 | 172.27.86.65 | 172.27.86.65 |
| SR-BODEGA | 172.27.86.128 255.255.255.224 | 172.27.86.129 | 172.27.86.129 |
| SR-VENTAS | 172.27.86.160 255.255.255.224 | 172.27.86.161 | 172.27.86.161 |
| SR-GERENCIA | 172.27.86.192 255.255.255.224 | 172.27.86.193 | 172.27.86.193 |
| SR-CONTABILIDAD | 172.27.86.224 255.255.255.240 | 172.27.86.225 | 172.27.86.225 |
| PZ-BODEGA | 172.27.87.0 255.255.255.224 | 172.27.87.1 | 172.27.87.1 |
| PZ-VENTAS | 172.27.87.32 255.255.255.224 | 172.27.87.33 | 172.27.87.33 |
| PZ-GERENCIA | 172.27.87.64 255.255.255.224 | 172.27.87.65 | 172.27.87.65 |
| PZ-CONTABILIDAD | 172.27.87.96 255.255.255.240 | 172.27.87.97 | 172.27.87.97 |

---

## D4 – Seguridad (Fase 5)

Aplicada y probada por Jose el 08/11/2026 en los 12 equipos (scripts `configs/<sede>/<EQUIPO>/<EQUIPO>-fase5.txt`). Para revalidar antes de entregar: [checklist de entrega](docs/checklist-entrega.md), sección 1. Cumple los puntos 4 y 7 del enunciado: banner, contraseñas, descripciones y SSH como único acceso, administrado por exactamente 3 usuarios.

### Credenciales (iguales en los 12 equipos)

> ⚠️ El repositorio debe seguir **privado**: aquí están las contraseñas.

| Uso | Comando | Valor |
| --- | --- | --- |
| Administrador 1 | `username kendal privilege 15 secret` | `redes` |
| Administrador 2 | `username mariangel privilege 15 secret` | `redes` |
| Administrador 3 | `username sebas privilege 15 secret` | `redes` |
| Modo privilegiado | `enable secret` | `proyecto` |
| Consola | `line console 0` → `login local` | Pide usuario y contraseña de la tabla |

### Configuración común

| Bloque | Comandos |
| --- | --- |
| Base | `no ip domain-lookup` · `ip domain-name clavitoveloz.com` · `service password-encryption` |
| Banner | `banner motd "ACCESO RESTRINGIDO, SOLO PERSONAL AUTORIZADO - EL CLAVITO VELOZ"` |
| Consola | `login local` · `logging synchronous` · `exec-timeout 5 0` |
| VTY 0–15 | `login local` · `transport input ssh` · `exec-timeout 5 0` |
| SSH | `crypto key generate rsa` (2048 bits) · `ip ssh version 2` |

### Acceso SSH por sede

Desde la laptop admin de cada sede (Desktop → Command Prompt): `ssh -l kendal <IP>` → contraseña `redes`. Telnet debe fallar.

| Sede | Laptop admin | Router | SW01 | SW02 | SW03 |
| --- | --- | --- | --- | --- | --- |
| Grecia | GRE-ADM-LT01 (.86.117) | 172.27.86.113 | 172.27.86.114 | 172.27.86.115 | 172.27.86.116 |
| San Ramón | SR-ADM-LT01 (.86.245) | 172.27.86.241 | 172.27.86.242 | 172.27.86.243 | 172.27.86.244 |
| Pérez Zeledón | PZ-ADM-LT01 (.87.117) | 172.27.87.113 | 172.27.87.114 | 172.27.87.115 | 172.27.87.116 |

### Puertos apagados (`shutdown`)

| Switch | Puertos sin uso apagados |
| --- | --- |
| GRE-SW01 | Fa0/23 |
| GRE-SW02 | Fa0/21–24 |
| GRE-SW03 | Gi0/2 (Fa0/1–24 todos en uso) |
| SR-SW01 / PZ-SW01 | Fa0/21–23 |
| SR-SW02 / PZ-SW02 | Fa0/17–24 |
| SR-SW03 / PZ-SW03 | Fa0/23–24 y Gi0/2 |

Routers: Gi0/0 (LAN), Gi0/1 y Gi0/2 (WAN) en uso; no quedan interfaces libres.

---

## Fase 4 – PCs

Aplicada por Jose el 09/11/2026 en `pkt/ClavitoVeloz.pkt` (respaldo `pkt/versiones/fase4-PoblarPCs.pkt`): las 182 PCs completas según la tabla, DHCP en todos los departamentos excepto Gerencia Grecia (8 PCs estáticas .98–.105). Se hizo directamente en Packet Tracer, sin scripts.

| Sede | Departamento | PCs requeridos | Switch / puertos | IP |
| --- | --- | --- | --- | --- |
| Grecia | Ventas | 22 | GRE-SW01 Fa0/1–22 | DHCP |
| Grecia | Producción | 20 | GRE-SW02 Fa0/1–20 | DHCP |
| Grecia | Desarrollo | 16 | GRE-SW03 Fa0/1–16 | DHCP |
| Grecia | Gerencia | 8 | GRE-SW03 Fa0/17–24 | Estática .98–.105 /28, GW .97 |
| Sucursales | Bodega | 20 | XX-SW01 Fa0/1–20 | DHCP |
| Sucursales | Ventas | 16 | XX-SW02 Fa0/1–16 | DHCP |
| Sucursales | Gerencia | 12 | XX-SW03 Fa0/1–12 | DHCP |
| Sucursales | Contabilidad | 10 | XX-SW03 Fa0/13–22 | DHCP |

---

## Topología física (Fase 0)

Armada en Packet Tracer el 26/10/2026 con 1 PC por departamento + 1 laptop admin por sede (los 182 PCs se agregan en la Fase 4). Troncales y WAN según [D3](#d3--puertos-conexiones-e-ips-fijas).

### Equipos finales colocados

| Sede | Equipo (Display Name) | Tipo | Switch | Puerto | VLAN |
| --- | --- | --- | --- | --- | --- |
| Grecia | GRE-VEN-PC01 | PC-PT | GRE-SW01 | Fa0/1 | 300 |
| Grecia | GRE-ADM-LT01 | Laptop-PT | GRE-SW01 | Fa0/24 | 900 |
| Grecia | GRE-PRO-PC01 | PC-PT | GRE-SW02 | Fa0/1 | 200 |
| Grecia | GRE-DES-PC01 | PC-PT | GRE-SW03 | Fa0/1 | 400 |
| Grecia | GRE-GER-PC01 | PC-PT | GRE-SW03 | Fa0/17 | 100 |
| San Ramón | SR-BOD-PC01 | PC-PT | SR-SW01 | Fa0/1 | 500 |
| San Ramón | SR-ADM-LT01 | Laptop-PT | SR-SW01 | Fa0/24 | 900 |
| San Ramón | SR-VEN-PC01 | PC-PT | SR-SW02 | Fa0/1 | 300 |
| San Ramón | SR-GER-PC01 | PC-PT | SR-SW03 | Fa0/1 | 100 |
| San Ramón | SR-CON-PC01 | PC-PT | SR-SW03 | Fa0/13 | 600 |
| Pérez Zeledón | PZ-BOD-PC01 | PC-PT | PZ-SW01 | Fa0/1 | 500 |
| Pérez Zeledón | PZ-ADM-LT01 | Laptop-PT | PZ-SW01 | Fa0/24 | 900 |
| Pérez Zeledón | PZ-VEN-PC01 | PC-PT | PZ-SW02 | Fa0/1 | 300 |
| Pérez Zeledón | PZ-GER-PC01 | PC-PT | PZ-SW03 | Fa0/1 | 100 |
| Pérez Zeledón | PZ-CON-PC01 | PC-PT | PZ-SW03 | Fa0/13 | 600 |

Nomenclatura de equipos finales: `SEDE-DEPTO-PC##` (VEN, PRO, DES, GER, BOD, CON) y `SEDE-ADM-LT##` para la laptop admin.

### Presentación del lienzo

- Las 3 sedes en triángulo, cada una en un rectángulo de color con su título (`SEDE - GRECIA`, `SEDE - SAN RAMON`, `SEDE - PEREZ ZELEDON`).
- Nota por sede con sus VLANs, subredes y gateways.
- Esperado sin configuración: enlaces de los routers en rojo (interfaces apagadas); switches y PCs en verde.

---

## Problemas encontrados y solución

Registro de fallas reales durante la construcción (sirve de repaso para la comprobación individual).

| Fecha | Fase | Síntoma | Diagnóstico | Solución |
| --- | --- | --- | --- | --- |
| 09/11 | 4–5 | `git pull` falla: *Your local changes to pkt/ClavitoVeloz.pkt would be overwritten* | Packet Tracer había vuelto a guardar el .pkt local (sin cambios reales) y el compañero subió una versión nueva | Cerrar Packet Tracer, `git restore pkt\ClavitoVeloz.pkt` y `git pull`. Regla: no dejar el .pkt modificado sin commit al terminar |
| 07/11 | 2 | Ping de SR-BOD-PC01 a Grecia: `Reply from 172.27.86.129: Destination host unreachable` | La respuesta viene del gateway propio → la LAN de SR funciona, pero SR-RT01 no tiene ruta a Grecia. `show ip interface brief` en SR-RT01: Gi0/2 **up/down** (cable conectado, el otro extremo apagado). En GRE-RT01: Gi0/1 `administratively down` sin IP | Faltaba aplicar `GRE-RT01-fase2.txt` (IP WAN + `no shutdown` + EIGRP). Al aplicarlo se formó el vecino EIGRP y aparecieron las rutas `D` |
| 03/11 | 1 | Los comandos de Grecia "no aparecían" en el repo | Se escribieron dentro de los `.gitkeep` (y el de SW03 en la carpeta de la sede) | Movidos a `configs/grecia/<EQUIPO>/<EQUIPO>-fase1.txt` con encabezado; regla agregada en `configs/README.md` |

Comandos de diagnóstico que más sirvieron: `show ip interface brief` (status/protocol), `show cdp neighbors` (qué puerto conecta con cuál), `show ip eigrp neighbors`, `show ip route`.

---

## Pendientes

### Para la próxima sesión

- [ ] **Confirmar la prueba de respaldo** (opcional +10): `tracert 172.27.86.2` desde PZ-BOD-PC01, apagar GRE-RT01 Gi0/2, repetir y ver que pasa por 172.27.87.133. Volver a encender.
- [ ] **Nombre del archivo final:** el enunciado pide `NOMBRE-NOMBRE-NOMBRE.PKT` (3 nombres) y el grupo es de 5. Confirmar con el profesor si van los 5 nombres.
- [ ] **Guía de estudio** para la comprobación individual (5 pts).

### Confirmaciones del grupo

- [ ] **EIGRP AS 10** como protocolo definitivo (D5.1, en uso y funcionando).
- [ ] **Versión de Packet Tracer** de todos (la misma).

### Trabajo por hacer

- [ ] **Fase 6 – Cierre:** recorrer el [checklist de entrega](docs/checklist-entrega.md) el 25/11 (incluye la revisión completa de SSH en los 12 equipos).
- [ ] **Respaldos de configuración real:** `show running-config` de cada equipo en su carpeta de `configs/`.
- [ ] **Capturas de pruebas — Mariangel.** Guardar `.png` en `pruebas/<sede>/` según `pruebas/README.md`. Mínimo:
    - Grecia: ping de GRE-VEN-PC01 a Producción (.34), Desarrollo (.66), Gerencia (.98) y laptop (.117); `ipconfig` de un PC con DHCP; `show vlan brief` y `show interfaces trunk` de un switch.
    - San Ramón: lo mismo con SR-BOD-PC01 (.162, .194, .226, .245).
    - Pérez Zeledón: ping de PZ-BOD-PC01 a .87.34, .87.66, .87.98 y .87.117.
    - WAN: ping entre sedes; `show ip eigrp neighbors` y `show ip route` en los 3 routers.
    - Respaldo (opcional +10): `tracert` antes y después de apagar el enlace PZ–GRE.
    - **Seguridad:** SSH exitoso desde cada laptop admin (se ve el banner), `telnet` rechazado, `show ip ssh`.

---

## Comandos de referencia (vistos en clase)

Los laboratorios originales están en [`docs/apuntes-clase/`](docs/apuntes-clase/).

| Tema | Comandos | Uso en el proyecto |
| --- | --- | --- |
| VLANs y acceso | `vlan X` · `name` · `interface range` · `switchport mode access` · `switchport access vlan X` | Una VLAN por departamento |
| Troncal | `switchport mode trunk` · `switchport trunk native vlan 100` (ambos extremos) | Enlaces switch–switch y switch–router |
| Router-on-a-stick | `interface gi0/0.X` · `encapsulation dot1Q X [native]` · `ip address` | Gateway de cada VLAN |
| DHCP | `ip dhcp excluded-address` · `ip dhcp pool` · `network` · `default-router` · `dns-server` | Pools en el router de cada sede |
| SSH | `ip domain-name` · `crypto key generate rsa` (2048) · `username … privilege 15 secret …` · `line vty 0 15` · `login local` · `transport input ssh` · `ip ssh version 2` | Acceso único, 3 usuarios |
| Gestión del switch | `interface vlan 900` · `ip address` · `ip default-gateway` | SVI en VLAN admin |
| Hardening | `hostname` · `no ip domain-lookup` · `enable secret` · `service password-encryption` · `banner motd #…#` · `line console 0` · `logging synchronous` · `exec-timeout 5 0` · `description` | Todos los equipos |
| Guardar / verificar | `wr` · `show vlan brief` · `show ip interface brief` · `show ip route` · `show running-config` | Cierre y pruebas |

> Lección del laboratorio de troncales: la VLAN nativa debe coincidir en los dos extremos del trunk; si no, aparece `NATIVE_VLAN_MISMATCH` y el puerto se bloquea.

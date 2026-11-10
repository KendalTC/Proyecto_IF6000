# Checklist de entrega final – Proyecto IF6000

Revisar sobre el `.pkt` final **el día antes de la entrega (miércoles 25/11/2026)**. Marcar cada casilla en GitHub o en el editor. Si algo falla, anotarlo en la sección "Problemas encontrados" del README.

Credenciales: usuarios `kendal`, `mariangel`, `sebas` · clave `redes` · enable `proyecto`.

---

## 1. SSH – acceso remoto (punto 7 del enunciado)

Desde la laptop admin de cada sede → **Desktop → Command Prompt** → `ssh -l kendal <IP>` → clave `redes`.
Cada casilla se marca solo si: pide la clave, **muestra el banner** y entra al equipo.

| Sede | Laptop | Equipo | IP | SSH entra | Banner | `telnet` rechazado |
| --- | --- | --- | --- | --- | --- | --- |
| Grecia | GRE-ADM-LT01 | GRE-RT01 | 172.27.86.113 | [ ] | [ ] | [ ] |
| Grecia | GRE-ADM-LT01 | GRE-SW01 | 172.27.86.114 | [ ] | [ ] | [ ] |
| Grecia | GRE-ADM-LT01 | GRE-SW02 | 172.27.86.115 | [ ] | [ ] | [ ] |
| Grecia | GRE-ADM-LT01 | GRE-SW03 | 172.27.86.116 | [ ] | [ ] | [ ] |
| San Ramón | SR-ADM-LT01 | SR-RT01 | 172.27.86.241 | [ ] | [ ] | [ ] |
| San Ramón | SR-ADM-LT01 | SR-SW01 | 172.27.86.242 | [ ] | [ ] | [ ] |
| San Ramón | SR-ADM-LT01 | SR-SW02 | 172.27.86.243 | [ ] | [ ] | [ ] |
| San Ramón | SR-ADM-LT01 | SR-SW03 | 172.27.86.244 | [ ] | [ ] | [ ] |
| Pérez Zeledón | PZ-ADM-LT01 | PZ-RT01 | 172.27.87.113 | [ ] | [ ] | [ ] |
| Pérez Zeledón | PZ-ADM-LT01 | PZ-SW01 | 172.27.87.114 | [ ] | [ ] | [ ] |
| Pérez Zeledón | PZ-ADM-LT01 | PZ-SW02 | 172.27.87.115 | [ ] | [ ] | [ ] |
| Pérez Zeledón | PZ-ADM-LT01 | PZ-SW03 | 172.27.87.116 | [ ] | [ ] | [ ] |

Además:

- [ ] Los **otros 2 usuarios** también entran (probar `mariangel` y `sebas` al menos en un equipo por sede).
- [ ] **SSH entre sedes:** desde GRE-ADM-LT01, `ssh -l kendal 172.27.86.241` (SR-RT01) y `ssh -l kendal 172.27.87.113` (PZ-RT01).
- [ ] En cada equipo: `show ip ssh` → **SSH Enabled - version 2.0**.
- [ ] En cada equipo: `show running-config | include username` → **exactamente 3 líneas**.
- [ ] La **consola** de cada equipo pide usuario y clave.
- [ ] `enable` pide la clave `proyecto`.

## 2. Hardening (punto 4)

- [ ] Los 12 equipos tienen su **hostname** (GRE-RT01, SR-SW02…).
- [ ] **Banner** visible al entrar por consola y por SSH.
- [ ] **Descripciones** en interfaces: `show interfaces description` en un router y un switch por sede.
- [ ] **Puertos sin uso apagados:** `show interfaces status` → "disabled" (GRE-SW01 Fa0/23 · GRE-SW02 Fa0/21–24 · GRE-SW03 Gi0/2 · XX-SW01 Fa0/21–23 · XX-SW02 Fa0/17–24 · XX-SW03 Fa0/23–24 y Gi0/2).
- [ ] `show running-config` no muestra contraseñas en texto plano.

## 3. VLANs y troncales

- [ ] `show vlan brief` en los 9 switches: VLANs correctas y puertos en su VLAN.
- [ ] `show interfaces trunk` en los 9 switches: trunks con **native 100**, sin `NATIVE_VLAN_MISMATCH`.
- [ ] `show ip interface brief` en los 3 routers: Gi0/0, sus subinterfaces, Gi0/1 y Gi0/2 en **up/up**.

## 4. Direccionamiento (DHCP / estático)

- [ ] Una PC de cada departamento con DHCP recibe IP de **su** subred (Desktop → IP Configuration, nunca 169.254.x.x).
- [ ] `show ip dhcp binding` en los 3 routers muestra IPs entregadas.
- [ ] Gerencia Grecia: 8 PCs estáticas .98–.105, máscara 255.255.255.240, GW 172.27.86.97.
- [ ] Laptops admin estáticas: .86.117 / .86.245 / .87.117 (máscara /29).

## 5. Conectividad (30 pts)

- [ ] **Dentro de cada sede:** ping entre todas las VLANs (una PC por departamento).
- [ ] **Entre sedes:** ping desde una PC de cada sede a una PC de las otras dos.
- [ ] `show ip eigrp neighbors` en cada router → **2 vecinos**.
- [ ] `show ip route` en cada router → rutas **D** hacia las otras dos sedes.
- [ ] **Respaldo (opcional +10):** `tracert 172.27.86.2` desde PZ-BOD-PC01 → apagar GRE-RT01 Gi0/2 → repetir `tracert` (pasa por 172.27.87.133) → **volver a encender** Gi0/2.

## 6. Presentación (15 pts)

- [ ] Las 3 sedes con rectángulo de color y título.
- [ ] Notas de VLAN/subred/gateway correctas en cada sede.
- [ ] Notas de los 3 enlaces WAN.
- [ ] Todos los cables en verde (excepto puertos apagados a propósito).
- [ ] Nombres de PCs y equipos visibles y sin "(1)".

## 7. Archivo y repositorio

- [ ] Archivo final con el nombre que pide el enunciado (`NOMBRE-NOMBRE-…​.PKT`) en `pkt/`.
- [ ] Abre en la versión de Packet Tracer del laboratorio.
- [ ] Respaldo de cada fase en `pkt/versiones/`.
- [ ] Capturas de pruebas en `pruebas/` (Mariangel).
- [ ] Todo con commit y push; repositorio privado.
- [ ] Cada integrante repasó la guía de estudio para la comprobación individual.

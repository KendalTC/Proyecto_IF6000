# Guía de estudio y guion de demostración – Proyecto IF6000

Para la **comprobación grupal e individual (5 pts)** y la **demostración de pruebas (30 pts)** del jueves 26/11/2026.
Cada integrante debe poder explicar **cualquier** parte, no solo la que configuró.

---

## 1. Guion de demostración (orden sugerido, ~10 min)

| # | Qué mostrar | Dónde | Rubro |
| --- | --- | --- | --- |
| 1 | Vista general: 3 sedes, triángulo WAN, notas de cada sede | Lienzo | Topología 5 · Cableado 5 |
| 2 | Tabla VLSM: red base 172.27.86.0/23, un /25 por sede, /30 en WAN | README / nota | Subneteo 5 |
| 3 | `show vlan brief` y `show interfaces trunk` en un switch | GRE-SW03 | VLANs |
| 4 | `show ip interface brief` y `show run` (subinterfaces, DHCP) | GRE-RT01 | Interfaces, DHCP |
| 5 | `ipconfig` en una PC DHCP y una estática (Gerencia Grecia) | GRE-VEN-PC01, GRE-GER-PC01 | Asignación IP |
| 6 | Ping entre VLANs dentro de Grecia | GRE-VEN-PC01 | Pruebas |
| 7 | Ping a San Ramón y Pérez Zeledón | GRE-VEN-PC01 | Pruebas |
| 8 | `show ip eigrp neighbors` y `show ip route` | PZ-RT01 | Enrutamiento |
| 9 | SSH desde la laptop admin (banner + entra) y telnet rechazado | GRE-ADM-LT01 | SSH / Seguridad |
| 10 | Respaldo: `tracert`, apagar GRE Gi0/2, `tracert` otra vez, encender | PZ-BOD-PC01 + GRE-RT01 | Opcional +10 |

Tener abierto el .pkt **antes** de que llegue el profesor y hacer un ping previo a cada destino (el primero pierde paquetes por ARP).

---

## 2. Conceptos que hay que dominar

### Subneteo VLSM
- **Red base:** 172.27.86.0/23 → 512 direcciones (172.27.86.0 – 172.27.87.255). Se eligió una red privada clase B poco común para no coincidir con otros grupos.
- **Método:** ordenar subredes de mayor a menor y asignar el bloque más pequeño que alcance. Hosts necesarios = PCs + gateway (+20 % de crecimiento).
- **Fórmula:** hosts usables = 2^(32 − prefijo) − 2.

| Prefijo | Máscara | Bloque | Usables | Uso en el proyecto |
| --- | --- | --- | --- | --- |
| /25 | 255.255.255.128 | 128 | 126 | Un bloque por sede |
| /27 | 255.255.255.224 | 32 | 30 | Ventas, Producción, Desarrollo, Bodega, Gerencia sucursales |
| /28 | 255.255.255.240 | 16 | 14 | Gerencia Grecia (8 PCs), Contabilidad (10) |
| /29 | 255.255.255.248 | 8 | 6 | VLAN 900 admin (router + 3 SW + laptop = 5) |
| /30 | 255.255.255.252 | 4 | 2 | Enlaces WAN punto a punto |

- **Wildcard** = 255.255.255.255 − máscara. /23 → 0.0.1.255 (por eso `network 172.27.86.0 0.0.1.255` cubre toda la red en EIGRP).
- **"Sin desperdicio":** /29 para admin (6 usables, se usan 5); /30 en WAN (2 usables, 2 usados).

Ejercicio típico: *"¿Cuál es el broadcast de Ventas San Ramón?"* → 172.27.86.160/27 → bloque de 32 → broadcast .191.

### VLANs y troncales
- **VLAN:** divide un switch en dominios de broadcast separados. Una por departamento, mismo ID en todas las sedes (100 Gerencia, 200 Producción, 300 Ventas, 400 Desarrollo, 500 Bodega, 600 Contabilidad, 900 Admin).
- **Puerto de acceso:** lleva una sola VLAN, sin etiqueta (PCs).
- **Troncal 802.1Q:** lleva varias VLANs etiquetadas (switch–switch, switch–router).
- **VLAN nativa (punto 6):** viaja **sin etiqueta** en el trunk. Elegimos **100 GERENCIA**. Debe coincidir en los dos extremos; si no → `NATIVE_VLAN_MISMATCH`.
- En el router: `encapsulation dot1Q 100 native`.

### Router-on-a-stick
- Un solo cable físico (Gi0/0) del router al switch, dividido en **subinterfaces** (Gi0/0.100, .300…). Cada subinterfaz es el **gateway** de su VLAN.
- Por qué: ahorra puertos del router (enunciado pide austeridad). El 2911 tiene 3 Gigabit: 1 LAN + 2 WAN.

### DHCP (punto 5)
- Departamentos con **10 o más** PCs → DHCP. Gerencia Grecia (8) → **estática**. Sucursales → DHCP obligatorio en todos los departamentos.
- Comandos: `ip dhcp excluded-address` (antes del pool) · `ip dhcp pool` · `network` · `default-router`.
- Se excluye el gateway para que el pool no lo entregue a una PC.
- VLAN 900 es de infraestructura (router, switches, laptop admin), por eso es estática: los equipos de gestión deben tener IP fija para poder entrar por SSH.
- Si una PC sale con 169.254.x.x → no llegó al servidor DHCP (VLAN mal asignada, trunk caído o pool mal escrito).

### Enrutamiento entre sedes – EIGRP AS 10
- Protocolo dinámico de Cisco (vector distancia avanzado). Métrica por ancho de banda + retardo. Distancia administrativa interna **90** (estática 1, OSPF 110, RIP 120).
- `router eigrp 10` · `network 172.27.86.0 0.0.1.255` · `no auto-summary` (evita que resuma a la clase B 172.27.0.0/16 y rompa el VLSM).
- El AS debe ser **igual** en los 3 routers para ser vecinos.
- En `show ip route`, las rutas aprendidas salen con **D**; las directamente conectadas con **C**.

### Rutas de respaldo (opcional +10)
- **Triángulo WAN:** cada sede tiene 2 caminos hacia cada otra sede (directo y pasando por la tercera).
- EIGRP elige el camino de menor métrica (el directo) y guarda el otro como **feasible successor**. Si cae el enlace, cambia **automáticamente** en segundos.

### Seguridad y SSH (puntos 4 y 7)
- **VLAN 900 ADMIN-SSH:** "área administrativa de seguridad". Los switches tienen IP en `interface vlan 900` y `ip default-gateway`.
- **SSH requiere:** hostname distinto de "Router/Switch" + `ip domain-name` → `crypto key generate rsa` (2048) → usuarios locales → `line vty 0 15` con `login local` y `transport input ssh` → `ip ssh version 2`.
- **SSH vs Telnet:** SSH cifra la sesión (puerto 22); Telnet va en texto plano (puerto 23). `transport input ssh` bloquea telnet.
- **3 administradores** (exactamente, en todos): `kendal`, `mariangel`, `sebas`, privilege 15.
- `enable secret` (hash MD5) vs `enable password` (texto plano / cifrado débil). `service password-encryption` cifra las demás contraseñas en el `show run`.
- `banner motd` · `exec-timeout 5 0` (cierra la sesión tras 5 min) · `shutdown` en puertos sin uso (evita que alguien conecte un equipo no autorizado).

### Cableado
- **Directo:** dispositivos distintos (PC–switch, switch–router).
- **Cruzado:** dispositivos iguales (switch–switch, router–router).
- Switches en cadena: con N switches 2960-24TT quedan 24N + 1 puertos para equipos finales (cada enlace entre switches usa los Gigabit). 3 switches por sede porque 2 no alcanzan para 67 equipos en Grecia.

---

## 3. Preguntas probables (con respuesta)

1. **¿Por qué 172.27.86.0/23?** Privada (rango 172.16–31), poco común, y 512 direcciones alcanzan las 372 necesarias con reserva.
2. **¿Por qué Gerencia Grecia es /28 y la de sucursal /27?** 8 PCs + GW = 9 → /28 (14). Sucursal 12 PCs + 20 % + GW ≈ 16 → no cabe en /28 → /27.
3. **¿Cuál es la VLAN nativa y por qué?** 100 GERENCIA; el enunciado pide escoger un segmento como nativo y Gerencia existe en las 3 sedes.
4. **¿Qué pasa si la nativa no coincide?** Mensaje `NATIVE_VLAN_MISMATCH`, el tráfico sin etiqueta cae en la VLAN equivocada.
5. **¿Cómo sabe una PC de San Ramón llegar a Grecia?** Envía al gateway (subinterfaz); el router consulta su tabla, tiene ruta **D** aprendida por EIGRP y la envía por Gi0/2.
6. **¿Por qué el switch necesita `ip default-gateway`?** Para responder SSH a una laptop de otra subred/sede.
7. **¿Qué hace `no auto-summary`?** Anuncia cada subred con su máscara real; sin él, EIGRP resumiría a clase B y habría conflictos.
8. **¿Qué pasa si se cae el enlace PZ–GRE?** EIGRP recalcula y el tráfico pasa por San Ramón (demostrarlo con `tracert`).
9. **¿Cuántos equipos se compraron?** 2 routers, 7 switches, 159 computadoras (ver inventario en el README).
10. **¿Por qué las IPs excluidas son solo los gateways?** Los demás equipos con IP fija (switches, laptop) están en la VLAN 900, que no tiene pool.
11. **¿Cómo comprobar que SSH está activo?** `show ip ssh` → *SSH Enabled - version 2.0*.
12. **¿Cómo ver qué IPs entregó el DHCP?** `show ip dhcp binding`.

---

## 4. Diagnóstico rápido (si algo falla en la demo)

| Síntoma | Comando | Causa típica |
| --- | --- | --- |
| PC con 169.254.x.x | `show vlan brief` en el switch | Puerto en otra VLAN o trunk caído |
| *Destination host unreachable* desde el propio gateway | `show ip route` | Falta ruta (vecino EIGRP caído) |
| *Request timed out* | `show ip interface brief` | Interfaz `down` o `administratively down` |
| Sin vecinos EIGRP | `show ip eigrp neighbors` | Interfaz WAN apagada, IP mal, AS distinto |
| SSH: *Connection refused* | `show ip ssh` · `show run \| section vty` | Faltan llaves RSA o `transport input` |
| Ping falla la primera vez | — | Normal: resolución ARP |

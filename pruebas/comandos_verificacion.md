# Verificación final en Packet Tracer (Fase 6)

> **Nota (26/11):** este es el instructivo que se usó el 22/11. Las salidas no quedaron en `pruebas/salidas/`: la configuración real se guardó como `configs/<sede>/<EQUIPO>/<EQUIPO>-running-config.txt` y los resultados en las capturas de `pruebas/`.

Archivo a abrir: `pkt/ClavitoVeloz.pkt` (es la versión más reciente: Fase 4 sobre Fase 5).
Credenciales: usuario `kendal` · clave `redes` · enable `proyecto`.

Se hace en **3 rondas**. Al terminar cada ronda, guardar lo indicado y avisar para revisarlo.

---

## Ronda 1 – Sacar la configuración real de los 12 equipos

Esto permite revisar la configuración completa sin abrir el .pkt, y de paso deja el respaldo de `show running-config` que pide `configs/README.md`.

**Cómo:** clic en el equipo → pestaña **CLI** → Enter → usuario `kendal`, clave `redes` (entra directo a `#` porque el usuario es privilege 15).
Pegar el bloque completo. Luego seleccionar todo el texto de la CLI (clic dentro, `Cmd+A`, `Cmd+C`) y pegarlo en un archivo:

`pruebas/salidas/<EQUIPO>.txt` → por ejemplo `pruebas/salidas/GRE-RT01.txt`

> Si la CLI corta el texto, subir el **buffer** no se puede en PT; en ese caso copiar en dos partes (primero el `show running-config`, después el resto).

### Routers (GRE-RT01, SR-RT01, PZ-RT01)

```

show running-config
show ip interface brief
show ip eigrp neighbors
show ip route
show ip dhcp binding
show ip ssh
show interfaces description
```

### Switches (los 9: GRE-SW01…03, SR-SW01…03, PZ-SW01…03)

```

show running-config
show vlan brief
show interfaces trunk
show interfaces status
show ip ssh
```

Lo que se va a revisar con eso:

| Punto | Esperado |
| --- | --- |
| `show ip interface brief` (router) | Gi0/0, Gi0/0.100…900, Gi0/1, Gi0/2 en **up / up** |
| `show ip eigrp neighbors` | **2 vecinos** en cada router |
| `show ip route` | Rutas **D** a las 2 otras sedes (las 10 subredes remotas) |
| `show ip dhcp binding` | IPs entregadas en cada pool |
| `show vlan brief` | Cada puerto en su VLAN según D3 del README |
| `show interfaces trunk` | Gi0/1 y Gi0/2 en trunking, **native 100** |
| `show interfaces status` | Puertos sin uso en **disabled** |
| `show ip ssh` | **SSH Enabled - version 2.0** |
| `show running-config` | 3 `username`, `enable secret`, `service password-encryption`, banner, `transport input ssh` |

---

## Ronda 2 – Pruebas desde PCs (conectividad, DHCP, SSH)

En cada PC: clic → **Desktop → Command Prompt**. Tomar captura (`Cmd+Shift+4`) de cada resultado exitoso y guardarla en `pruebas/<sede>/` con el nombre indicado.

### 2.1 DHCP – una PC por departamento

`ipconfig` en cada una. Debe salir una IP de **su** subred, nunca `169.254.x.x`.
Si alguna sale vacía: Desktop → IP Configuration → cambiar a Static y volver a **DHCP**.

| PC | Debe recibir | Captura |
| --- | --- | --- |
| GRE-VEN-PC01 | 172.27.86.2 – .30 | `grecia/dhcp-GRE-VEN.png` |
| GRE-PRO-PC01 | 172.27.86.34 – .62 | `grecia/dhcp-GRE-PRO.png` |
| GRE-DES-PC01 | 172.27.86.66 – .94 | `grecia/dhcp-GRE-DES.png` |
| GRE-GER-PC01 | **estática** 172.27.86.98 /28 GW .97 | `grecia/estatica-GRE-GER.png` |
| SR-BOD-PC01 · SR-VEN-PC01 · SR-GER-PC01 · SR-CON-PC01 | .86.130+ · .86.162+ · .86.194+ · .86.226+ | `san-ramon/dhcp-SR-XXX.png` |
| PZ-BOD-PC01 · PZ-VEN-PC01 · PZ-GER-PC01 · PZ-CON-PC01 | .87.2+ · .87.34+ · .87.66+ · .87.98+ | `perez-zeledon/dhcp-PZ-XXX.png` |

### 2.2 Ping entre VLANs dentro de cada sede

Hacer primero `ipconfig` en el destino para saber su IP real (DHCP puede no dar exactamente la del ejemplo). El primer ping puede perder 1 paquete (ARP): repetir.

```
GRE-VEN-PC01>  ping <IP GRE-PRO-PC01>   ping <IP GRE-DES-PC01>   ping 172.27.86.98   ping 172.27.86.117
SR-BOD-PC01>   ping <IP SR-VEN-PC01>    ping <IP SR-GER-PC01>    ping <IP SR-CON-PC01>   ping 172.27.86.245
PZ-BOD-PC01>   ping <IP PZ-VEN-PC01>    ping <IP PZ-GER-PC01>    ping <IP PZ-CON-PC01>   ping 172.27.87.117
```

Captura: `<sede>/ping-entre-vlans.png`

### 2.3 Ping entre sedes

```
GRE-VEN-PC01>  ping <IP SR-BOD-PC01>     ping <IP PZ-BOD-PC01>
SR-BOD-PC01>   ping 172.27.86.98         ping <IP PZ-VEN-PC01>
PZ-BOD-PC01>   ping 172.27.86.98         ping <IP SR-CON-PC01>
```

Captura: `wan/ping-entre-sedes-<origen>.png`

### 2.4 SSH (punto 7 del enunciado)

Desde la laptop admin de cada sede, a los 4 equipos de su sede:

```
GRE-ADM-LT01>  ssh -l kendal 172.27.86.113     (luego .114, .115, .116)
SR-ADM-LT01>   ssh -l kendal 172.27.86.241     (luego .242, .243, .244)
PZ-ADM-LT01>   ssh -l kendal 172.27.87.113     (luego .114, .115, .116)
```

Debe pedir clave (`redes`), **mostrar el banner** y entrar con `#`. Salir con `exit`.
Probar también una vez `ssh -l mariangel …` y `ssh -l sebas …`.

**Telnet debe fallar:** `telnet 172.27.86.113` → *Connection refused* / se cierra.

**SSH entre sedes:** desde GRE-ADM-LT01 → `ssh -l kendal 172.27.86.241` y `ssh -l kendal 172.27.87.113`.

Capturas: `<sede>/ssh-<EQUIPO>.png` (al menos router + 1 switch por sede), `grecia/telnet-rechazado.png`, `wan/ssh-entre-sedes.png`.

---

## Ronda 3 – Ruta de respaldo (opcional, +10 pts)

1. En **PZ-BOD-PC01**: `tracert 172.27.86.98` → debe salir por **172.27.87.138** (enlace directo PZ → GRE).
   Captura: `wan/respaldo-1-antes.png`
2. En **GRE-RT01** (CLI):
   ```
   configure terminal
   interface GigabitEthernet0/2
    shutdown
   end
   ```
3. Esperar ~10 s. En PZ-BOD-PC01: `tracert 172.27.86.98` → ahora pasa por **172.27.87.133 → 172.27.87.129** (vía San Ramón). Un `ping -t` durante el corte muestra que se recupera solo.
   Captura: `wan/respaldo-2-enlace-caido.png`. Opcional: `show ip route` en PZ-RT01 (`wan/respaldo-3-ruta.png`).
4. **Volver a encender** (muy importante antes de guardar):
   ```
   configure terminal
   interface GigabitEthernet0/2
    no shutdown
   end
   write memory
   ```
5. Repetir el `tracert`: vuelve a salir por .87.138.

---

## Al terminar

- En **cada** equipo que se haya tocado: `write memory` (si no, al cerrar el .pkt se pierde).
- Guardar el .pkt (`Cmd+S`).
- Copiarlo con el nombre de entrega (ver "Archivo final" en el README).

# Lab 01 — Inter-VLAN Routing (Router-on-a-Stick)

## Escenario
Red corporativa simulada con tres departamentos (Administración, Ventas y TI)
que requieren segmentación lógica pero comunicación controlada entre sí.

## Objetivo
Segmentar el tráfico en tres VLANs y habilitar routing entre ellas usando
una sola interfaz física del router (técnica router-on-a-stick con 802.1Q).

## Herramientas
- Cisco Packet Tracer 8.x
- Router Cisco 2901
- Switch Cisco 2960
- 3 PCs genéricas

## Topología

| Dispositivo | VLAN | Subred           | IP asignada     | Gateway       |
|-------------|------|------------------|-----------------|---------------|
| PC-Admin    | 10   | 192.168.10.0/24  | 192.168.10.10   | 192.168.10.1  |
| PC-Ventas   | 20   | 192.168.20.0/24  | 192.168.20.10   | 192.168.20.1  |
| PC-TI       | 30   | 192.168.30.0/24  | 192.168.30.10   | 192.168.30.1  |

## Configuración clave

### Switch — VLANs y puertos
vlan 10
name Administracion
vlan 20
name Ventas
vlan 30
name TI

interface fastEthernet0/1
switchport mode access
switchport access vlan 10

interface fastEthernet0/24
switchport mode trunk
switchport trunk allowed vlan 10,20,30

### Router — subinterfaces 802.1Q

interface gigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface gigabitEthernet0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

## Problemas encontrados
Como fue uno de mis primeros labs, la mayor dificultad fue por desconocimiento de configuración del router, ya despues de orientarme un poco, se me hizo fácil. 

## Verificación
- `ping 192.168.10.1` desde PC-Admin → Reply (TTL=255, mismo dispositivo)
- `ping 192.168.20.10` desde PC-Admin → Reply (TTL=127, pasó por el router ✓)

## Resultado
Inter-VLAN routing funcional. Las tres VLANs se comunican a través del router
conservando segmentación lógica independiente en cada segmento de red.

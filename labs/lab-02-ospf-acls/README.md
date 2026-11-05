# Lab 02 — OSPF y ACLs

## Escenario
Dos redes locales conectadas por dos routers. Los routers deben aprender
las rutas automáticamente (OSPF) y se debe bloquear el ping de PC-A hacia
PC-B sin afectar el resto del tráfico (ACL extendida).

## Herramientas
- Cisco Packet Tracer
- 2 routers, 2 PCs

## Direccionamiento

| Dispositivo | Interfaz | IP | Conecta a |
|---|---|---|---|
| R1 | Gi0/0/0 | 10.0.1.1/24 | PC-A |
| R1 | Gi0/0/1 | 192.168.100.1/30 | R2 |
| R2 | Gi0/0/0 | 192.168.100.2/30 | R1 |
| R2 | Gi0/0/1 | 10.0.2.1/24 | PC-B |
| PC-A | NIC | 10.0.1.10/24 | GW 10.0.1.1 |
| PC-B | NIC | 10.0.2.10/24 | GW 10.0.2.1 |

## Configuración clave

### OSPF (R1)
```
router ospf 1
 router-id 1.1.1.1
 network 10.0.1.0 0.0.0.255 area 0
 network 192.168.100.0 0.0.0.3 area 0
```

### ACL (R1)
```
ip access-list extended BLOQUEAR-PING-A-B
 deny icmp 10.0.1.0 0.0.0.255 host 10.0.2.10 echo
 permit ip any any

interface gigabitEthernet0/0/0
 ip access-group BLOQUEAR-PING-A-B in
```

## Verificación
- `show ip ospf neighbor`: R2 en estado FULL.
- `show ip route`: ruta `O 10.0.2.0/24 [110/2]` aprendida por OSPF.
- Ping PC-A → PC-B con TTL=126 antes de aplicar la ACL (cruzó 2 routers).
- Ping PC-A → PC-B bloqueado después de aplicar la ACL.
- `show access-lists`: contador de coincidencias en la regla deny.

## Problemas encontrados y aprendizajes
- **Primer ping con timeout:** el primer paquete se pierde por el proceso ARP
  (resolución de la MAC del gateway). Los siguientes responden normal.
- **Nombres de interfaz:** el modelo de router usa Gi0/0/x en vez de Gi0/x,
  por lo que hubo que adaptar los comandos.
- **Deny implícito:** toda ACL termina con un deny all invisible. Sin el
  `permit ip any any` final, se bloquearía todo el tráfico de la interfaz,
  incluido OSPF.

## Resultado
OSPF forma adyacencia FULL entre R1 y R2 y las rutas se aprenden de forma
dinámica. La ACL bloquea únicamente el ICMP de PC-A hacia PC-B.

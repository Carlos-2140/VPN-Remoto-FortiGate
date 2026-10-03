# Direccionamiento IP del laboratorio

Este documento consolida el direccionamiento utilizado en la topología **VPN Remoto FortiGate**.

## 1. Red física y salida a Internet

| Equipo | Interfaz | IP | Prefijo | Gateway |
|---|---|---:|---:|---:|
| PC física | NIC LAN | 192.168.18.40 | /24 | 192.168.18.1 |
| R1 | FastEthernet1/0 | 192.168.18.167 | /24 | 192.168.18.1 |

La interfaz `FastEthernet1/0` de R1 se conecta a **Cloud1** y permite que la infraestructura virtual utilice la red física como salida a Internet.

## 2. Enlace R1 - R2

**Red:** `198.51.100.0/30`

| Equipo | Interfaz | IP |
|---|---|---:|
| R1 | FastEthernet0/0 | 198.51.100.1 |
| R2 | FastEthernet0/0 | 198.51.100.2 |

Este segmento funciona como enlace de tránsito entre el router ISP R1 y el router R2 de la red de usuarios.

## 3. Enlace R1 - FortiGate

**Red:** `203.0.113.0/30`

| Equipo | Interfaz | IP |
|---|---|---:|
| R1 | FastEthernet0/1 | 203.0.113.1 |
| FortiGate | port1 | 203.0.113.2 |

`port1` es la interfaz WAN del FortiGate y también el punto de terminación de la VPN IPsec de acceso remoto.

## 4. Red de usuarios

**Red:** `10.21.40.0/25`  
**Máscara:** `255.255.255.128`

| Equipo | Interfaz | IP | Función |
|---|---|---:|---|
| R2 | FastEthernet1/0 | 10.21.40.1 | Gateway |
| Windows10-Usuario-1 | Ethernet | 10.21.40.21 | Cliente DHCP |

El router R2 proporciona DHCP a los clientes de la VLAN 10. Se reservaron las primeras direcciones de la red para infraestructura.

## 5. Red del servidor

**Red:** `10.21.40.128/28`  
**Máscara:** `255.255.255.240`

| Equipo | Interfaz | IP | Función |
|---|---|---:|---|
| FortiGate | port2 | 10.21.40.129 | Gateway |
| Ubuntu Server 24.04 | ens3 | 10.21.40.130 | HTTPS / SSH |

El servidor utiliza como gateway la interfaz `port2` del FortiGate.

## 6. Pool de clientes VPN

| Parámetro | Valor |
|---|---|
| Rango asignable | `10.21.50.10 - 10.21.50.20` |
| Máscara | `255.255.255.0` |
| Red lógica | `10.21.50.0/24` |
| Red accesible por split tunnel | `10.21.40.128/28` |
| Host SSH autorizado | `10.21.40.130/32` |

Cuando FortiClient establece la VPN, recibe una dirección dentro de este rango y una ruta hacia la red protegida del servidor.

## 7. DNS utilizado por los clientes

Los servidores DNS configurados para el laboratorio son:

- `8.8.8.8`
- `1.1.1.1`

## 8. Resumen del flujo de tráfico

### HTTPS sin VPN

```text
Windows10-Usuario-1
10.21.40.21
      |
      v
R2
10.21.40.1
      |
      v
R1
      |
      v
FortiGate
203.0.113.2
      |
      v
Ubuntu Server
10.21.40.130:443
```

### SSH mediante VPN IPsec

```text
Windows10-Usuario-1
      |
      | VPN IPsec / FortiClient
      v
FortiGate
VPN_REMOTE_SSH
      |
      | TCP/22
      v
Ubuntu Server
10.21.40.130
```

La política del FortiGate permite HTTPS desde la red de usuarios, mientras que SSH se autoriza únicamente desde la interfaz del túnel VPN.

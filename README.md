# VPN Remoto FortiGate

Laboratorio de **VPN IPsec de acceso remoto con FortiGate en GNS3**.

El objetivo principal es permitir acceso **HTTPS** al servidor sin necesidad de VPN y restringir el acceso **SSH** para que solo funcione cuando el cliente se conecta mediante la VPN IPsec.

## Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Máscara / Prefijo | Gateway / Siguiente salto | Función |
|---|---|---:|---:|---:|---|
| PC física | NIC LAN | 192.168.18.40 | /24 | 192.168.18.1 | Administración desde la red física |
| R1 | FastEthernet1/0 | 192.168.18.167 | /24 | 192.168.18.1 | Salida hacia Cloud1 / Internet |
| R1 | FastEthernet0/0 | 198.51.100.1 | /30 | — | Enlace R1 ↔ R2 |
| R2 | FastEthernet0/0 | 198.51.100.2 | /30 | 198.51.100.1 | Enlace R2 ↔ R1 |
| R1 | FastEthernet0/1 | 203.0.113.1 | /30 | — | Enlace R1 ↔ FortiGate |
| FortiGate | port1 | 203.0.113.2 | /30 | 203.0.113.1 | WAN / acceso VPN |
| R2 | FastEthernet1/0 | 10.21.40.1 | /25 | — | Gateway VLAN 10 / usuarios |
| Windows10-Usuario-1 | Ethernet | 10.21.40.21 | /25 | 10.21.40.1 | Cliente del laboratorio |
| FortiGate | port2 | 10.21.40.129 | /28 | — | Gateway de la red del servidor |
| Ubuntu Server 24.04 | ens3 | 10.21.40.130 | /28 | 10.21.40.129 | Servidor HTTPS y SSH |

### Pool de clientes VPN

- **Rango VPN:** `10.21.50.10 - 10.21.50.20`
- **Máscara:** `255.255.255.0 (/24)`
- **Red protegida:** `10.21.40.128/28`
- **Servidor protegido:** `10.21.40.130/32`

## Redes utilizadas

| Red | Uso |
|---|---|
| `192.168.18.0/24` | Red física / administración / salida a Internet |
| `198.51.100.0/30` | Tránsito R1 ↔ R2 |
| `203.0.113.0/30` | Tránsito R1 ↔ FortiGate |
| `10.21.40.0/25` | Red de usuarios / VLAN 10 |
| `10.21.40.128/28` | Red del servidor |
| `10.21.50.0/24` | Pool lógico de clientes VPN |

## Servicios del servidor

| Servicio | Puerto | Acceso esperado |
|---|---:|---|
| HTTPS | TCP/443 | Permitido sin VPN |
| SSH | TCP/22 | Permitido únicamente mediante VPN IPsec |

## Estado actual del laboratorio

- Conectividad entre R1, R2 y FortiGate: configurada.
- VLAN 10 y DHCP para el cliente Windows: configurados.
- Ubuntu Server con `10.21.40.130/28`: configurado.
- Apache HTTPS: funcionando.
- OpenSSH Server: funcionando.
- HTTPS sin VPN: funcionando.
- SSH sin VPN: bloqueado por política de FortiGate.
- VPN IPsec de acceso remoto: funcionando.
- Cliente VPN recibe una IP del pool `10.21.50.10-20`.
- SSH mediante la VPN hacia `10.21.40.130`: funcionando.

## Documentación

La tabla ampliada de direccionamiento está disponible en:

- [Direccionamiento/Direccionamiento_IP.md](Direccionamiento/Direccionamiento_IP.md)

> **Nota de seguridad:** no se publican contraseñas, claves precompartidas (PSK) ni otros secretos en este repositorio.

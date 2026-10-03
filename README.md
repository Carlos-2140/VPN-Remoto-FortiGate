# VPN Remoto con FortiGate

> **Video demostrativo:** [Agregar enlace del video aquí](#)

Laboratorio de **VPN IPsec de acceso remoto con FortiGate en GNS3**, diseñado para demostrar segmentación de red, control de acceso y acceso remoto seguro a un servidor Ubuntu.

El escenario permite que un usuario acceda al servicio **HTTPS** del servidor sin utilizar la VPN, mientras que el servicio **SSH** únicamente queda disponible cuando el cliente establece el túnel **IPsec con FortiClient**.

---

## Objetivos

- Implementar una topología funcional en **GNS3**.
- Configurar un **FortiGate** como firewall y terminador VPN.
- Proporcionar una red de usuarios mediante **VLAN 10 y DHCP**.
- Publicar un servidor **Ubuntu Server 24.04** con servicios HTTPS y SSH.
- Permitir **HTTPS sin VPN**.
- Bloquear **SSH directo** desde la red de usuarios.
- Permitir **SSH únicamente mediante la VPN IPsec**.
- Documentar el direccionamiento, las configuraciones y las pruebas realizadas.

---

## Topología

La infraestructura está compuesta por:

```text
                    Cloud1 / Internet
                           |
                           |
                          R1
                    _____/  \_____
                   /              \
                  /                \
                R2              FortiGate
                 |              port1: 203.0.113.2
              Switch                |
                 |               port2
         Windows 10                 |
         Usuario VLAN 10       Ubuntu Server
         10.21.40.21           10.21.40.130
```

El cliente Windows se conecta a la red de usuarios a través de R2. El FortiGate protege la red del servidor y actúa como punto de terminación de la VPN IPsec.

La documentación gráfica de la topología se encuentra en:

- [Topologia](Topologia)

---

## Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Prefijo | Función |
|---|---|---:|---:|---|
| PC física | NIC LAN | `192.168.18.40` | /24 | Administración |
| R1 | FastEthernet1/0 | `192.168.18.167` | /24 | Salida hacia Cloud1 |
| R1 | FastEthernet0/0 | `198.51.100.1` | /30 | Tránsito R1 ↔ R2 |
| R2 | FastEthernet0/0 | `198.51.100.2` | /30 | Tránsito R2 ↔ R1 |
| R1 | FastEthernet0/1 | `203.0.113.1` | /30 | Tránsito R1 ↔ FortiGate |
| FortiGate | port1 | `203.0.113.2` | /30 | WAN / VPN |
| R2 | FastEthernet1/0 | `10.21.40.1` | /25 | Gateway VLAN 10 |
| Windows10-Usuario-1 | Ethernet | `10.21.40.21` | /25 | Cliente DHCP |
| FortiGate | port2 | `10.21.40.129` | /28 | Gateway del servidor |
| Ubuntu Server 24.04 | ens3 | `10.21.40.130` | /28 | HTTPS / SSH |

### Redes utilizadas

| Red | Uso |
|---|---|
| `192.168.18.0/24` | Administración y salida a Internet |
| `198.51.100.0/30` | Tránsito R1 ↔ R2 |
| `203.0.113.0/30` | Tránsito R1 ↔ FortiGate |
| `10.21.40.0/25` | Usuarios / VLAN 10 |
| `10.21.40.128/28` | Red protegida del servidor |
| `10.21.50.0/24` | Pool lógico de clientes VPN |

Más detalles:

- [Direccionamiento/Direccionamiento_IP.md](Direccionamiento/Direccionamiento_IP.md)

---

## VPN IPsec

La VPN de acceso remoto fue configurada en FortiGate con FortiClient.

| Parámetro | Valor |
|---|---|
| Nombre | `VPN_REMOTE_SSH` |
| Tipo | Remote Access / Dial-up |
| Interfaz WAN | `port1` |
| Red protegida | `10.21.40.128/28` |
| Servidor SSH | `10.21.40.130` |
| Pool VPN | `10.21.50.10 - 10.21.50.20` |
| Usuario VPN | `vpnuser` |
| Grupo | `VPN_USERS` |
| Split Tunnel | Habilitado |

### Parámetros utilizados en el laboratorio

**Phase 1**

- IKEv1
- Aggressive Mode
- DES / SHA256
- DH Group 14
- Lifetime: 86400 segundos

**Phase 2**

- DES / SHA1
- PFS habilitado
- DH Group 14
- Lifetime: 43200 segundos

> **Nota:** DES se utilizó únicamente por compatibilidad con el entorno de laboratorio. En ambientes de producción se deben emplear algoritmos criptográficos modernos.

La documentación del FortiGate se encuentra en:

- [Configuracion fortigate](Configuracion%20fortigate)

---

## Servidor Ubuntu

El servidor utiliza:

```text
Sistema: Ubuntu Server 24.04
IP:      10.21.40.130/28
Gateway: 10.21.40.129
```

Servicios instalados:

| Servicio | Puerto | Uso |
|---|---:|---|
| Apache HTTPS | TCP/443 | Acceso web |
| OpenSSH | TCP/22 | Administración remota mediante VPN |

La configuración automatizada del servidor está disponible en:

- [Script](Script)

---

## Políticas de seguridad

El FortiGate aplica dos comportamientos principales:

### HTTPS sin VPN

```text
Windows 10
10.21.40.21
     |
     | TCP/443
     v
R2 → R1 → FortiGate → Ubuntu Server
                       10.21.40.130
```

**Resultado:** permitido.

### SSH sin VPN

```text
Windows 10
     |
     | TCP/22
     v
FortiGate
     X
Ubuntu Server
```

**Resultado:** bloqueado.

### SSH mediante VPN

```text
Windows 10
     |
     | FortiClient / VPN IPsec
     v
FortiGate
VPN_REMOTE_SSH
     |
     | TCP/22
     v
Ubuntu Server
10.21.40.130
```

**Resultado:** permitido.

---

## Validaciones realizadas

| Prueba | Resultado |
|---|---|
| Cliente obtiene IP por DHCP | ✅ |
| Gateway de VLAN 10 accesible | ✅ |
| Conectividad R1 ↔ R2 | ✅ |
| Conectividad R1 ↔ FortiGate | ✅ |
| HTTPS hacia Ubuntu sin VPN | ✅ |
| SSH hacia Ubuntu sin VPN | ❌ Bloqueado |
| Establecimiento de VPN IPsec | ✅ |
| Asignación de IP del pool VPN | ✅ |
| SSH mediante VPN | ✅ |
| Split Tunnel hacia red del servidor | ✅ |

---

## Estructura del repositorio

```text
VPN-Remoto-FortiGate/
│
├── README.md
├── Configuracion fortigate/
├── Configuracion switch/
├── Direccionamiento/
│   └── Direccionamiento_IP.md
├── Script/
├── Show running-config/
└── Topologia/
```

### Contenido

- **Configuracion fortigate/**  
  Evidencias y documentación de la configuración realizada desde la GUI de FortiGate.

- **Configuracion switch/**  
  Configuración relacionada con el segmento de usuarios y VLAN 10.

- **Direccionamiento/**  
  Tabla y explicación detallada del direccionamiento IPv4.

- **Script/**  
  Scripts utilizados para configurar el servidor Ubuntu.

- **Show running-config/**  
  Configuraciones de los dispositivos de red.

- **Topologia/**  
  Diagramas de la infraestructura implementada.

---

## Tecnologías utilizadas

- GNS3
- FortiGate VM
- FortiClient
- Cisco IOS
- Ubuntu Server 24.04
- Apache2
- OpenSSH
- IPsec
- VLAN
- DHCP
- HTTPS
- SSH

---

## Seguridad del repositorio

Este repositorio **no debe incluir**:

- contraseñas reales;
- claves precompartidas de la VPN;
- credenciales de usuarios;
- secretos o información sensible.

Las claves deben sustituirse en la documentación por valores como:

```text
<PSK_REDACTED>
<PASSWORD_REDACTED>
```

---

## Autor

**Carlos**  
Laboratorio de infraestructura y seguridad de redes.

---

## Estado del proyecto

**Laboratorio funcional.**

- HTTPS sin VPN: ✅
- SSH directo: bloqueado ✅
- VPN IPsec: ✅
- SSH mediante VPN: ✅
- Documentación: en proceso
- Video demostrativo: pendiente de agregar

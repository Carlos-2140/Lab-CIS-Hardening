# Direccionamiento IP — Laboratorio CIS Hardening (GNS3)

Este documento registra el direccionamiento utilizado en la topología del laboratorio. Las direcciones corresponden a las configuraciones y pruebas realizadas.

## Tabla de direccionamiento

| Equipo | Interfaz | Dirección / máscara | Puerta de enlace o siguiente salto | Función |
|---|---|---|---|---|
| GNS3 NAT | Red externa | 192.168.42.0/24 | 192.168.42.1 | Red NAT del entorno GNS3 |
| Router Cisco **R-CIS** | FastEthernet0/1 | DHCP; observada: **192.168.42.214/24** | 192.168.42.1 | WAN hacia GNS3 NAT; NAT outside |
| Router Cisco **R-CIS** | FastEthernet0/0 | **10.21.40.1/30** (255.255.255.252) | — | Tránsito al FortiGate; NAT inside |
| FortiGate 7.0.9 | port1 | **10.21.40.2/30** (255.255.255.252) | Ruta predeterminada vía 10.21.40.1 | WAN hacia R-CIS |
| FortiGate 7.0.9 | port2 | **10.21.40.129/25** (255.255.255.128) | — | Gateway de la LAN |
| Switch Cisco **SW-CIS** | VLAN 1 (gestión) | **10.21.40.130/25** | 10.21.40.129 | Administración del switch |
| Ubuntu Server 24.04.5 LTS | ens3 | **10.21.40.138/25** | 10.21.40.129 | Servidor de automatización Python / Paramiko |
| Windows 10 QEMU | NIC LAN | **10.21.40.140/25** | 10.21.40.129 | Estación de pruebas |

> **Nota:** La WAN del router utiliza DHCP. `192.168.42.214` es la dirección observada durante el laboratorio, no una IP fija garantizada.

## Segmentos de red

| Segmento | Red | Máscara | Rango de hosts utilizable | Broadcast |
|---|---|---|---|---|
| Tránsito R-CIS ↔ FortiGate | 10.21.40.0/30 | 255.255.255.252 | 10.21.40.1–10.21.40.2 | 10.21.40.3 |
| LAN de hardening | 10.21.40.128/25 | 255.255.255.128 | 10.21.40.129–10.21.40.254 | 10.21.40.255 |
| Red NAT de GNS3 | 192.168.42.0/24 | 255.255.255.0 | 192.168.42.1–192.168.42.254 | 192.168.42.255 |

## Recorrido lógico

```text
GNS3 NAT (192.168.42.1)
    |
    | 192.168.42.0/24 (DHCP)
    |
R-CIS Fa0/1 (192.168.42.214 observada)
R-CIS Fa0/0 (10.21.40.1/30)
    |
    | Tránsito 10.21.40.0/30
    |
FortiGate port1 (10.21.40.2/30)
FortiGate port2 (10.21.40.129/25)
    |
    | LAN 10.21.40.128/25
    |
SW-CIS VLAN 1 (10.21.40.130/25)
    |-- Ubuntu Server ens3 (10.21.40.138/25)
    |-- Windows 10 QEMU (10.21.40.140/25)
```

## Rutas, NAT y DNS

- **Router R-CIS:** ruta estática `10.21.40.128/25` vía `10.21.40.2`. NAT overload hacia FastEthernet0/1 para la red interna.
- **FortiGate:** ruta predeterminada `0.0.0.0/0` vía `10.21.40.1` por `port1`.
- **SW-CIS, Ubuntu y Windows 10:** puerta de enlace `10.21.40.129`.
- **Ubuntu Server:** DNS `8.8.8.8` y `1.1.1.1`; IP estática configurada mediante `/etc/netplan/50-cloud-init.yaml`.
- **Administración del laboratorio:** el firewall conserva SSH y HTTP en la LAN mientras se investiga el fallo de negociación HTTPS; la administración remota en `port1` está restringida a `ping`.

## Comprobaciones realizadas

Se efectuaron pruebas de conectividad ICMP y de accesibilidad TCP/22 hacia los tres dispositivos. Los resultados y evidencias se documentan en los informes de auditoría y capturas del laboratorio.

*Documento de direccionamiento de la topología CIS Hardening en GNS3.*

# Laboratorio CIS Hardening en GNS3
### Auditoría y fortalecimiento de seguridad en Router Cisco, Switch Cisco y FortiGate

> 🎥 **VIDEO DE DEMOSTRACIÓN — PENDIENTE DE PUBLICACIÓN**  
> En esta primera sección se colocará el enlace real del video de YouTube (máximo **10 minutos**, con rostro, voz, fecha y hora según las instrucciones del profesor). No se ha inventado un enlace ni se afirma que el video esté publicado.

**Laboratorio académico | GNS3 | Cisco IOS 12.4 / IOSvL2 15.2 | FortiOS 7.0.9 | Ubuntu Server 24.04 | Python y Paramiko**

---

## Contenido

1. [Introducción y alcance](#1-introducción-y-alcance)
2. [Objetivos y metodología](#2-objetivos-y-metodología)
3. [Topología de GNS3](#3-topología-de-gns3)
4. [Direccionamiento y rutas](#4-direccionamiento-y-rutas)
5. [Herramientas utilizadas](#5-herramientas-utilizadas)
6. [Preparación de Ubuntu Server](#6-preparación-de-ubuntu-server)
7. [Controles CIS seleccionados: Router R-CIS](#7-controles-cis-seleccionados-router-r-cis)
8. [Controles CIS seleccionados: Switch SW-CIS](#8-controles-cis-seleccionados-switch-sw-cis)
9. [Controles CIS seleccionados: FortiGate](#9-controles-cis-seleccionados-fortigate)
10. [Cómo funcionan los scripts](#10-cómo-funcionan-los-scripts)
11. [Resultados reales de auditoría](#11-resultados-reales-de-auditoría)
12. [Pruebas de funcionamiento](#12-pruebas-de-funcionamiento)
13. [Evidencias visuales y documentación de GUI](#13-evidencias-visuales-y-documentación-de-gui)
14. [Configuraciones y archivos publicados](#14-configuraciones-y-archivos-publicados)
15. [Hallazgos, limitaciones y recomendaciones](#15-hallazgos-limitaciones-y-recomendaciones)
16. [Conclusiones](#16-conclusiones)
17. [Referencias técnicas](#17-referencias-técnicas)

## 1. Introducción y alcance

Se implementó un laboratorio de redes en **GNS3** para auditar y fortalecer la administración de **tres equipos de infraestructura**: router Cisco **R-CIS**, switch Cisco **SW-CIS** y firewall **FortiGate 7.0.9**. La automatización se realizó desde Ubuntu mediante **Python 3.12 y Paramiko 3.5.1**, utilizando conexiones SSH y generando reportes JSON de cumplimiento.

El trabajo se fundamenta en **principios de endurecimiento alineados con recomendaciones CIS**: reducir servicios administrativos innecesarios, exigir mecanismos de acceso seguros, limitar sesiones inactivas y fortalecer la autenticación. Cada resultado se documenta con el control inspeccionado, la configuración observada y el reporte real.

> **Alcance exacto:** Se escogieron **11 controles**, tres para router, tres para switch y cinco para FortiGate. Los porcentajes de este repositorio describen **solo estos controles**. No constituyen una certificación CIS ni una auditoría completa de ningún benchmark. No se atribuyen identificadores CIS concretos que no hayan sido comprobados contra la edición exacta y compatible con el software del laboratorio.

## 2. Objetivos y metodología

### Objetivo general

Auditar y reforzar los controles de acceso administrativo de la infraestructura GNS3, preservando la conectividad de los equipos, y demostrar objetivamente el estado de seguridad **antes y después**.

### Objetivos específicos

- Construir la topología con router, firewall, switch, Ubuntu Server y Windows 10.
- Identificar tres controles verificables en Cisco IOS y cinco controles en FortiOS.
- Ejecutar auditoría **sin cambios** mediante `--audit`.
- Evaluar y aplicar remediaciones mediante `--apply` cuando correspondiera.
- Generar y comparar reportes JSON auténticos.
- Verificar configuraciones mediante consola Cisco, CLI FortiGate y GUI FortiGate.
- Documentar una **excepción no resuelta** en lugar de marcarla como corregida.
- Consolidar los resultados y pruebas en un repositorio que pueda revisarse desde este README.

### Fases realizadas

| Fase | Actividad | Evidencia |
|---|---|---|
| 1. Implementación | Configuración de topología, interfaces, ruta predeterminada y acceso SSH | [Topología](Topologia/) y [Direccionamiento](Direccionamiento/) |
| 2. Auditoría inicial | Identificación de incumplimientos sin aplicar configuración | [JSON iniciales](Reportes/) |
| 3. Hardening | Ajuste de controles seleccionados en Cisco y FortiGate | [Scripts](Scripts/) y [configuraciones](Configuraciones/) |
| 4. Auditoría final | Comparación de controles cumplidos tras la intervención | [JSON finales](Reportes/) |
| 5. Validación | Pruebas SSH, revisión HTTP, acceso GUI y conectividad | [Evidencias PDF](Evidencias/Evidencias_CIS_Hardening.pdf) |
| 6. Documentación | Registro de procedimientos, resultados, excepción y referencias | Este README |

**Precisión:** Algunos cambios de FortiGate, especialmente el orden de activación de la política de contraseñas, requirieron una corrección manual. Los reportes demuestran los **estados observados** y no prueban que todos los cambios se ejecutaran automáticamente sin incidencias.

## 3. Topología de GNS3

### Diagrama lógico

![Diagrama del laboratorio CIS Hardening](Topologia/Diagrama.png)

### Captura de la topología real

![Topología real en GNS3](Topologia/Topologia.png)

### Recorrido entre dispositivos

```text
                  NAT1 — Red NAT GNS3
                          |
                       nat0
                          |
                 Fa0/1 (DHCP WAN)
                  ┌────────────┐
                  │ R-CIS      │ Cisco 2691 / IOS 12.4
                  └────────────┘
                 Fa0/0 10.21.40.1/30
                          |
                 port1 10.21.40.2/30
                ┌──────────────────┐
                │ FortiGate 7.0.9  │
                └──────────────────┘
                 port2 10.21.40.129/25
                          |
                        Gi0/0
                    ┌────────┐
                    │ SW-CIS │ Cisco IOSvL2
                    └────────┘
                     /      \
                 Gi0/2      Gi0/1
                   /          \
              Ubuntu Server   Windows 10 QEMU
              .138/25         .140/25
```

El servidor Ubuntu ejecuta los scripts contra el router, switch y firewall. El Windows 10 QEMU participa en las comprobaciones de acceso y funcionamiento. La topología utiliza **dos subredes internas**: tránsito router–firewall y LAN de usuarios/administración.

## 4. Direccionamiento y rutas

| Dispositivo | Interfaz | Dirección IP / máscara | Función |
|---|---|---|---|
| NAT1 | nat0 | Red `192.168.42.0/24` | NAT del entorno GNS3 |
| Router R-CIS | Fa0/1 | DHCP; **192.168.42.214/24 observado** | WAN, NAT outside |
| Router R-CIS | Fa0/0 | **10.21.40.1/30** | Enlace hacia FortiGate |
| FortiGate | port1 | **10.21.40.2/30** | WAN hacia router |
| FortiGate | port2 | **10.21.40.129/25** | Gateway LAN |
| Switch SW-CIS | Vlan1 | **10.21.40.130/25** | Gestión del switch |
| Ubuntu Server | ens3 | **10.21.40.138/25** | Automatización |
| Windows 10 QEMU | Ethernet | **10.21.40.140/25** | Pruebas |

**Red WAN NAT:** `192.168.42.0/24` (la IP del router se recibe por DHCP).  
**Red de tránsito:** `10.21.40.0/30` (hosts .1–.2).  
**Red LAN:** `10.21.40.128/25` (hosts .129–.254).

**Rutas relevantes:**
- Router: ruta estática hacia `10.21.40.128/25` a través de `10.21.40.2`.
- Router: NAT overload en `FastEthernet0/1` para la red interna definida en ACL 10.
- FortiGate: ruta predeterminada `0.0.0.0/0` a través de `10.21.40.1`, por `port1`.
- Ubuntu, switch y Windows: gateway `10.21.40.129`.

**Configuración de la LAN en Ubuntu (`/etc/netplan/50-cloud-init.yaml`):**

```yaml
network:
  version: 2
  ethernets:
    ens3:
      dhcp4: false
      addresses:
        - 10.21.40.138/25
      routes:
        - to: default
          via: 10.21.40.129
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Más información en [Direccionamiento/README.md](Direccionamiento/README.md).

## 5. Herramientas utilizadas

| Herramienta | Versión / detalle | Propósito |
|---|---|---|
| GNS3 | Laboratorio virtual | Topología y conexión de dispositivos |
| Cisco Router R-CIS | 2691 — IOS 12.4(25d) | Enrutamiento, NAT y hardening |
| Cisco Switch SW-CIS | IOSvL2 — IOS 15.2 | Conmutación y gestión segura |
| FortiGate | FortiOS 7.0.9 | Control de acceso y políticas |
| Ubuntu Server | 24.04.5 LTS | Servidor de automatización |
| Python | 3.12.3 | Scripts de auditoría |
| Paramiko | 3.5.1 | Conexión SSH automatizada |
| Windows 10 | QEMU dentro de GNS3 | Pruebas de conectividad y GUI |
| GitHub | Repositorio público | Presentación y trazabilidad |

## 6. Preparación de Ubuntu Server

Se configuró la dirección estática `10.21.40.138/25` con Netplan y el acceso de salida a través del FortiGate. Desde Ubuntu se instaló el entorno para ejecutar los scripts.

### Comprobación de red

```bash
ip -br addr show ens3
ip route
```

**Salida observada en el laboratorio:**

```text
ens3   UP   10.21.40.138/25
default via 10.21.40.129 dev ens3 proto static
10.21.40.128/25 dev ens3 proto kernel scope link src 10.21.40.138
```

### Entorno Python de la práctica

```bash
cd ~/CIS-Hardening-GNS3
source venv/bin/activate
python3 --version
python3 -m pip show paramiko
```

El laboratorio utilizó **Python 3.12.3** y **Paramiko 3.5.1** para establecer sesiones SSH con los dispositivos Cisco de versiones anteriores.

### Comprobación de los reportes guardados

```bash
cd ~/CIS-Hardening-GNS3
ls -lh reportes/
python3 -m json.tool reportes/router_antes.json
python3 -m json.tool reportes/router_despues.json
```

Estos comandos **leen reportes existentes**; mostrar un JSON no ejecuta una auditoría nueva. Los archivos anteriores y posteriores se han conservado en [Reportes](Reportes/). El detalle de preparación de Ubuntu se encuentra además en [scripts-servidor-ubuntu-cis-hardening.txt](Scripts/scripts-servidor-ubuntu-cis-hardening.txt).

## 7. Controles CIS seleccionados: Router R-CIS

**Equipo:** Cisco 2691 — IOS 12.4(25d) — IP de gestión/transición `10.21.40.1`.

### R1. Exigir SSH versión 2

- **Riesgo mitigado:** uso de protocolos inseguros u obsoletos para administración.
- **Estado inicial:** NO CUMPLE.
- **Configuración final:** `ip ssh version 2`, autenticación local y VTY restringidas a SSH.
- **Prueba:** `show ip ssh` y revisión de VTY.
- **Resultado final:** CUMPLE.

```cisco
conf t
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
end
```

### R2. Deshabilitar servidor HTTP

- **Riesgo mitigado:** interfaz administrativa HTTP sin protección TLS.
- **Estado inicial:** **CUMPLE** (el equipo ya tenía el servidor HTTP deshabilitado).
- **Configuración final:** `no ip http server` y `no ip http secure-server`.
- **Resultado final:** CUMPLE.

```cisco
show running-config | include ip http
```

**Salida final registrada:**

```text
no ip http server
no ip http secure-server
```

### R3. Timeout de inactividad en líneas VTY

- **Riesgo mitigado:** sesiones administrativas abiertas sin supervisión.
- **Estado inicial:** NO CUMPLE.
- **Configuración final:** `exec-timeout 5 0`.
- **Resultado final:** CUMPLE.

```cisco
conf t
line vty 0 4
 exec-timeout 5 0
 login local
 transport input ssh
end
show running-config | section line vty
```

**Resultado conjunto:** **1/3 → 3/3**.

**Evidencias:** [antes](Reportes/router_antes.json), [después](Reportes/router_despues.json), [running-config](Configuraciones/Show%20runnig-config%20R-CIS..txt), [script Python](Scripts/hardening_router.py.txt) e [informe visual](Evidencias/Evidencias_CIS_Hardening.pdf).

## 8. Controles CIS seleccionados: Switch SW-CIS

**Equipo:** Cisco IOSvL2 — IOS 15.2 — administración en `Vlan1: 10.21.40.130/25`.

### S1. Exigir SSH versión 2

- **Riesgo:** compatibilidad con versiones antiguas de acceso remoto.
- **Inicial:** NO CUMPLE.
- **Final:** `ip ssh version 2`, `login local` y `transport input ssh` en VTY — CUMPLE.

### S2. Deshabilitar servidor HTTP

- **Riesgo:** exposición de un servicio administrativo sin cifrar.
- **Inicial:** NO CUMPLE.
- **Final:** `no ip http server` — CUMPLE.
- **Precisión:** el switch mantiene `ip http secure-server`. El objetivo era retirar HTTP, no HTTPS.

**Salida observada:**

```text
no ip http server
ip http secure-server
```

### S3. Establecer timeout VTY

- **Riesgo:** reutilización de sesiones sin supervisión.
- **Inicial:** NO CUMPLE.
- **Final:** `exec-timeout 5 0` — CUMPLE.

**Fragmento de la configuración final:**

```cisco
ip ssh version 2
no ip http server
ip http secure-server
!
line vty 0 4
 exec-timeout 5 0
 login local
 transport input ssh
```

**Comandos de verificación:**

```cisco
show ip ssh
show running-config | include ip http
show running-config | section line vty
show ip interface brief
```

**Resultado conjunto:** **0/3 → 3/3**.

**Evidencias:** [antes](Reportes/switch_antes.json), [después](Reportes/switch_despues.json), [running-config](Configuraciones/Show%20runnig-config%20SW-CIS..txt), [script Python](Scripts/hardening_switch.py.txt) e [informe visual](Evidencias/Evidencias_CIS_Hardening.pdf).

## 9. Controles CIS seleccionados: FortiGate

**Equipo:** FortiGate VM — FortiOS 7.0.9. WAN `port1: 10.21.40.2/30`; LAN `port2: 10.21.40.129/25`.

### F1. Restringir servicios administrativos en WAN

- **Riesgo:** exponer administración HTTP/HTTPS/SSH en el lado externo.
- **Estado inicial:** `ping https ssh http` — NO CUMPLE.
- **Estado final:** solamente `ping` — CUMPLE.
- **GUI:** **Network → Interfaces → port1 → Edit**. Dirección `10.21.40.2/255.255.255.252`; en **Administrative Access** únicamente PING.
- **CLI de referencia:**

```fortios
config system interface
    edit "port1"
        set allowaccess ping
    next
end
```

### F2. Eliminar HTTP administrativo de la LAN (**pendiente**)

- **Riesgo:** el navegador transmite la administración HTTP sin TLS.
- **Situación observada:** port2 mantiene `ping https ssh http`.
- **Motivo:** se observó fallo de negociación TLS/HTTPS. Deshabilitar HTTP sin un acceso HTTPS funcional podía dejar inaccesible la GUI.
- **Resultado:** NO CUMPLE inicialmente y NO CUMPLE al final.
- **Plan:** corregir HTTPS y comprobar acceso; entonces retirar `http` de `allowaccess` y repetir la auditoría.

**No se ejecutará aquí una orden de deshabilitar HTTP**, porque el control está documentado como pendiente y la recuperación de HTTPS debe verificarse antes.

### F3. Timeout administrativo de cinco minutos

- **Riesgo:** sesiones administrativas abandonadas.
- **Estado inicial y final:** `admintimeout 5` — CUMPLE.
- **GUI:** **System → Settings → Idle timeout: 5**.
- **CLI de comprobación:** `get system global` o `show system global`.

### F4. Deshabilitar SSH versión 1

- **Riesgo:** permitir protocolos criptográficamente obsoletos.
- **Estado inicial y final:** `admin-ssh-v1 disable` — CUMPLE.
- **Comprobación:** consultar el valor global con la CLI de FortiGate.

### F5. Política de contraseñas de administración

- **Riesgo:** contraseñas débiles y susceptibles a adivinación.
- **Estado inicial:** deshabilitada — NO CUMPLE.
- **Estado final:** aplicada a `admin-password`, mínimo **12 caracteres**, al menos **1 mayúscula, 1 minúscula, 1 número y 1 símbolo** — CUMPLE.
- **GUI:** **System → Settings → Password Policy**; alcance **Admin**; requisitos de complejidad habilitados.

**Fragmento final de referencia:**

```fortios
config system password-policy
    set status enable
    set apply-to admin-password
    set minimum-length 12
    set min-upper-case-letter 1
    set min-lower-case-letter 1
    set min-number 1
    set min-non-alphanumeric 1
end
```

Durante la práctica fue necesario **habilitar primero la política** antes de asignar el alcance y sus parámetros; no se afirma que esa remediación fuera completamente automática.

**Otros parámetros documentados, fuera de los 5 controles evaluados:** la GUI mostró reutilización de contraseñas habilitada y expiración desactivada. No se cuentan en el resultado de 4/5.

### Ruta y política de salida

La **ruta predeterminada** fue `0.0.0.0/0` hacia `10.21.40.1` por `port1`.

La regla **LAN-A-INTERNET** usa origen `port2`, destino `port1`, origen y destino `all`, servicio `ALL`, horario `always`, acción `ACCEPT`, **NAT deshabilitado** y registro de tráfico habilitado. La traducción NAT se realiza en el router R-CIS. Esta regla pertenece a la conectividad del laboratorio; **no se contó como control de hardening de los cinco evaluados**.

**Resultado conjunto FortiGate:** **2/5 → 4/5**.

**Evidencias:** [antes](Reportes/fortigate_antes.json), [después](Reportes/fortigate_despues.json), [medición intermedia 3/5](Reportes/fortigate_antes_original.json), [configuración FortiGate](Configuraciones/Configuracion%20Fortigate.txt), [script](Scripts/hardening_fortigate.py.txt), [PDF de GUI](Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf).

## 10. Cómo funcionan los scripts

Los scripts se desarrollaron para conectarse a los dispositivos por SSH y evaluar únicamente los controles especificados.

### Modo `--audit`: solo lectura

```bash
cd ~/CIS-Hardening-GNS3
source venv/bin/activate

python3 hardening_router.py --audit
python3 hardening_switch.py --audit
python3 hardening_fortigate.py --audit
```

**Funcionamiento:** solicitar contraseña SSH, conectar, consultar parámetros, evaluar cada condición y generar un reporte JSON. No pretende cambiar el estado de los dispositivos.

### Modo `--apply`: remediación condicionada y comprobación

```bash
python3 hardening_router.py --apply
python3 hardening_switch.py --apply
python3 hardening_fortigate.py --apply
```

**Funcionamiento previsto:** inspeccionar la configuración, identificar controles no cumplidos, intentar ajustes aplicables y ejecutar una evaluación posterior.

**Advertencia:** `--apply` puede modificar la configuración real del dispositivo. No repetirlo sin revisar el código, comprobar acceso por consola y tener un respaldo. En FortiGate se documentó una incidencia de secuencia de comandos que exigió intervención manual.

### Ejemplo auténtico de mantenimiento del switch

En una **ejecución posterior al hardening**, el usuario ejecutó `python3 hardening_switch.py --apply`:

```text
Conectando a SW-CIS...
Conexion SSH establecida.
=== ESTADO INICIAL ===
SSH_V2: CUMPLE
HTTP_OFF: CUMPLE
TIMEOUT_VTY: CUMPLE
Controles cumplidos: 3/3
Cumplimiento seleccionado: 100.0%
No hay cambios pendientes.

=== ESTADO POSTERIOR ===
SSH_V2: CUMPLE
HTTP_OFF: CUMPLE
TIMEOUT_VTY: CUMPLE
Controles cumplidos: 3/3
Cumplimiento seleccionado: 100.0%
Todos los controles seleccionados cumplen.
```

Esto **demuestra una ejecución real y el comportamiento sin cambios pendientes**, pero **no debe confundirse con una captura de la aplicación inicial de remediaciones**.

Después, `python3 hardening_switch.py --audit` volvió a confirmar **3/3** sin modificar el switch.

### Publicación de scripts

Los archivos disponibles en GitHub son copias documentales `.py.txt`; los originales ejecutados en Ubuntu eran archivos `.py`. Algunos textos subidos comienzan con una línea de título que deberá retirarse antes de ejecutar el código como Python. También debe revisarse la lógica de reintentos SSH y tratamiento de errores para distribuir los scripts como ejecutables independientes.

**Código:** [router](Scripts/hardening_router.py.txt) · [switch](Scripts/hardening_switch.py.txt) · [FortiGate](Scripts/hardening_fortigate.py.txt) · [comandos Ubuntu](Scripts/scripts-servidor-ubuntu-cis-hardening.txt).

## 11. Resultados reales de auditoría

### Resumen consolidado

| Dispositivo | Antes | Después | Mejora |
|---|---:|---:|---:|
| Router R-CIS | **1/3 (33,33 %)** | **3/3 (100 %)** | +2 controles |
| Switch SW-CIS | **0/3 (0 %)** | **3/3 (100 %)** | +3 controles |
| FortiGate | **2/5 (40 %)** | **4/5 (80 %)** | +2 controles |
| **TOTAL** | **3/11 (27,3 %)** | **10/11 (90,9 %)** | **+7 controles** |

### Resultado por control

| Equipo / control | Antes | Después |
|---|---|---|
| Router — SSHv2 | No cumple | Cumple |
| Router — HTTP deshabilitado | Cumple | Cumple |
| Router — Timeout VTY | No cumple | Cumple |
| Switch — SSHv2 | No cumple | Cumple |
| Switch — HTTP deshabilitado | No cumple | Cumple |
| Switch — Timeout VTY | No cumple | Cumple |
| FortiGate — Acceso WAN restringido | No cumple | Cumple |
| FortiGate — HTTP LAN deshabilitado | **No cumple** | **No cumple** |
| FortiGate — Timeout admin | Cumple | Cumple |
| FortiGate — SSHv1 deshabilitado | Cumple | Cumple |
| FortiGate — Política de contraseñas | No cumple | Cumple |

### Reportes JSON originales

| Dispositivo | Antes | Después |
|---|---|---|
| R-CIS | [router_antes.json](Reportes/router_antes.json) | [router_despues.json](Reportes/router_despues.json) |
| SW-CIS | [switch_antes.json](Reportes/switch_antes.json) | [switch_despues.json](Reportes/switch_despues.json) |
| FortiGate | [fortigate_antes.json](Reportes/fortigate_antes.json) | [fortigate_despues.json](Reportes/fortigate_despues.json) |

La medición [fortigate_antes_original.json](Reportes/fortigate_antes_original.json) es **intermedia (3/5)** y no sustituye ni al estado inicial oficial de 2/5 ni al final de 4/5.

**Importante:** Los archivos locales de Ubuntu con sufijo `_antes.json` pueden sobrescribirse al ejecutar otra auditoría. Los JSON históricos publicados aquí representan la **medición inicial guardada en GitHub**.

## 12. Pruebas de funcionamiento

### 12.1 SSH hacia el router

Se ejecutó desde Ubuntu:

```bash
python3 test_ssh_router.py
```

**Resultado observado:** conexión SSH establecida; el dispositivo respondió con datos de **R-CIS, Cisco 2691, IOS 12.4(25d)**. Aunque el script mostraba un texto de «switch Cisco», la respuesta confirmó que se trataba del router R-CIS.

Esta prueba acredita una **sesión SSH funcional**, mientras que la exigencia específica de SSH versión 2 se sustenta además en la configuración y la auditoría.

### 12.2 Auditoría SSH del switch

```bash
python3 hardening_switch.py --audit
```

**Resultado real:** conexión SSH establecida, tres controles **CUMPLE**, **3/3** y mensaje de finalización **sin modificar el switch**.

### 12.3 HTTP en router y switch

Consultas ejecutadas desde la consola Cisco:

```cisco
R-CIS#show running-config | include ip http
```

```text
no ip http server
no ip http secure-server
```

```cisco
SW-CIS#show running-config | include ip http
```

```text
no ip http server
ip http secure-server
```

Estas pruebas **verifican la configuración administrativa** observada. No se presentan como una prueba independiente de conexión a TCP/80 o TCP/443.

### 12.4 Configuración GUI del FortiGate

Se obtuvieron capturas auténticas de:

1. **WAN port1:** `10.21.40.2/30` y acceso administrativo **PING** únicamente.
2. **LAN port2:** `10.21.40.129/25` y acceso administrativo `PING/HTTPS/SSH/HTTP`.
3. **System Settings:** timeout administrativo de **5 minutos**.
4. **Password Policy:** alcance **Admin**, mínimo **12**, requisitos de mayúscula/minúscula/número/símbolo.
5. **Static Routes:** ruta `0.0.0.0/0` hacia `10.21.40.1`.
6. **Firewall Policy:** regla `LAN-A-INTERNET` con acción `ACCEPT` y registro de tráfico.

Se verificó la preservación de las configuraciones de conectividad de la LAN mientras se aplicaban los controles seleccionados. La falla de HTTPS administrativo se documentó y **no se ocultó**.

## 13. Evidencias visuales y documentación de GUI

### Informes de capturas auténticas

**[Abrir todas las evidencias visuales del laboratorio (PDF)](Evidencias/Evidencias_CIS_Hardening.pdf)**

**[Abrir configuración gráfica del FortiGate (PDF)](Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf)**

**Relación de lo demostrado visualmente:**

| Grupo | Evidencia auténtica | Resultado |
|---|---|---|
| Topología | GNS3 real con NAT, router, FortiGate, switch y hosts | Topología implementada |
| Auditoría inicial del router | JSON `router_antes.json` abierto en Ubuntu | 1/3 |
| Auditoría final del router | JSON `router_despues.json` abierto en Ubuntu | 3/3 |
| Auditoría inicial del switch | JSON `switch_antes.json` abierto en Ubuntu | 0/3 |
| Auditoría final del switch | JSON `switch_despues.json` abierto en Ubuntu | 3/3 |
| Auditoría inicial del FortiGate | JSON `fortigate_antes.json` abierto en Ubuntu | 2/5 |
| Auditoría final del FortiGate | JSON `fortigate_despues.json` abierto en Ubuntu | 4/5 |
| Control router | Verificación `no ip http server` | Cumple |
| Control switch | Verificación `no ip http server` | Cumple |
| FortiGate GUI | WAN, LAN, política de contraseñas, timeout, ruta y política de tráfico | 4/5 controles auditados |
| SSH | Sesión desde Ubuntu al router | Conexión exitosa |
| Script `--audit` | Consulta al switch posterior al hardening | 3/3 sin cambios |
| Script `--apply` | Ejecución con controles previamente cumplidos | 3/3; sin cambios pendientes |

**Nota sobre las imágenes:** Este README muestra directamente las dos imágenes de topología ya publicadas. Las demás **capturas originales están reunidas en los PDF del repositorio** y no se presentan como imágenes independientes inexistentes. El texto y resultados esenciales de esas evidencias están reproducidos aquí para que la revisión de la metodología y conclusiones no dependa de descargar los PDF.

### Cómo leer correctamente las evidencias

- Las auditorías «antes» proceden de los **reportes históricos**.
- Las comprobaciones posteriores se hicieron sobre equipos **ya configurados**.
- La captura `--apply` del switch demuestra que el script detecta **«No hay cambios pendientes»**; no prueba que ese día remediara un incumplimiento.
- La captura del FortiGate final muestra explícitamente que **HTTP en la LAN sigue pendiente**.

## 14. Configuraciones y archivos publicados

| Recurso | Contenido | Enlace |
|---|---|---|
| Router final | Running-config con SSH, VTY, HTTP, NAT y rutas | [R-CIS](Configuraciones/Show%20runnig-config%20R-CIS..txt) |
| Switch final | Running-config de gestión, SSH, HTTP y VTY | [SW-CIS](Configuraciones/Show%20runnig-config%20SW-CIS..txt) |
| FortiGate | Configuración general de FortiOS 7.0.9 | [FortiGate](Configuraciones/Configuracion%20Fortigate.txt) |
| Scripts | Código Python documental para los tres dispositivos | [Scripts](Scripts/) |
| JSON | Reportes iniciales, finales e intermedio relevante | [Reportes](Reportes/) |
| Recomendaciones | Justificación de cada control y alcance | [Recomendaciones CIS](Documentacion/Recomendaciones_CIS.md) |
| Evidencias visuales | Documento PDF de capturas y pruebas | [Evidencias](Evidencias/Evidencias_CIS_Hardening.pdf) |
| GUI FortiGate | Capturas y descripciones de GUI | [Documento GUI](Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf) |
| Direccionamiento | Tabla y esquema de subredes | [Direccionamiento](Direccionamiento/) |
| Topología | Diagrama y captura GNS3 | [Topologia](Topologia/) |

**Seguridad de la información:** Las exportaciones completas de FortiGate pueden contener cadenas `ENC` con material sensible. El archivo histórico que se publicó requiere depuración de esos valores y revisión del historial de Git, idealmente con rotación de credenciales afectadas. El hecho de estar cifrados **no convierte esos datos en adecuados para un repositorio público**.

## 15. Hallazgos, limitaciones y recomendaciones

### Hallazgos comprobados

- Router y switch finalizaron con todos sus controles seleccionados cumplidos.
- FortiGate completó cuatro de cinco controles; la restricción de administración WAN y la política de contraseñas mejoraron.
- Los controles de timeout y SSHv1 del FortiGate **ya cumplían** al inicio.
- El acceso administrativo mediante SSH funcionó después del hardening.
- Las auditorías posteriores y la nueva ejecución de `--apply` del switch confirmaron el estado **3/3 sin cambios pendientes**.

### Limitación técnica principal

**El FortiGate no pudo completar el control «HTTP administrativo deshabilitado en LAN»** por el problema documentado de HTTPS/TLS. La situación no debe ocultarse con una marca de cumplimiento.

### Recomendaciones para una siguiente iteración

1. Resolver el acceso HTTPS del FortiGate, validar certificados y sincronización de tiempo/NTP; retirar HTTP de `port2` únicamente tras verificar el acceso seguro.
2. Incorporar controles CIS adicionales cuando exista correspondencia documentada con la versión exacta de IOS/FortiOS.
3. Revisar restricciones de origen administrativo, AAA, logging centralizado, NTP, banners y protección de credenciales.
4. Revisar contraseñas reutilizables y vigencia de contraseñas del FortiGate de acuerdo con una política institucional.
5. Publicar scripts directamente como `.py` **válidos y probados**, junto con dependencias y un procedimiento reproducible.
6. Nunca incluir contraseñas, hashes, claves privadas o valores cifrados de configuración en archivos públicos.

## 16. Conclusiones

El laboratorio logró elevar el cumplimiento de **3 de 11 a 10 de 11 controles seleccionados**, equivalente a **27,3 % → 90,9 %** del alcance académico definido. El router y el switch finalizaron en 100 % de sus respectivos controles; FortiGate llegó al 80 % y conservó **un control pendiente debidamente justificado**.

La práctica demostró el valor de evaluar las configuraciones **antes y después**, automatizar comprobaciones con Python/SSH y conservar reportes JSON y capturas como evidencia verificable. El procedimiento también evidenció una decisión operativa importante: **no sacrificar el acceso administrativo al firewall para marcar un control como corregido cuando HTTPS aún falla**.

La documentación central del proyecto se presenta **en este README**, atendiendo a la observación del profesor de no depender exclusivamente de archivos PDF. Los PDF se conservan como **anexos visuales**.

## 17. Referencias técnicas

- [Center for Internet Security — Catálogo oficial de CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks).
- [Fortinet — Documentación oficial de FortiOS](https://docs.fortinet.com/).
- [Fortinet — FortiOS 7.0 Best Practices](https://docs.fortinet.com/document/fortigate/7.0.0/best-practices).
- [Detalle de los controles y su correspondencia de buenas prácticas en este laboratorio](Documentacion/Recomendaciones_CIS.md).

---

**Proyecto académico desarrollado en GNS3.**  
**Pendiente de entrega:** agregar el enlace auténtico del video demostrativo al principio del README y crear el TXT final con el enlace del repositorio y del video.

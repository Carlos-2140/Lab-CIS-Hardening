# Laboratorio CIS Hardening en GNS3

> **Video demostrativo (pendiente de publicación):** aquí se insertará el enlace real de YouTube del laboratorio. La demostración deberá durar **máximo 10 minutos** e incluir rostro, voz, fecha y hora, según las instrucciones de la práctica. **Todavía no hay un enlace de video verificado.**

**Auditoría y fortalecimiento de seguridad de un router Cisco, un switch Cisco y un firewall FortiGate mediante Python, SSH y recomendaciones de hardening alineadas con CIS.**

![Diagrama de la topología CIS Hardening](Topologia/Diagrama.png)

## 1. Descripción del proyecto

Este laboratorio académico fue implementado en **GNS3** para comprobar la configuración de seguridad de dispositivos de infraestructura de red. Se diseñaron scripts Python que inspeccionan controles seleccionados mediante SSH, generan reportes JSON y permiten aplicar ajustes de hardening.

Los modos contemplados en los scripts son:

- `--audit`: consulta de controles sin modificar el dispositivo.
- `--apply`: evaluación, intento de aplicación de controles pendientes y comprobación posterior.

El proyecto documenta la **configuración observada**, las **auditorías iniciales y finales**, las **capturas auténticas de las pruebas**, la **topología** y las **excepciones pendientes**.

> **Nota de alcance:** Se evaluaron **11 controles seleccionados**. El porcentaje mostrado es exclusivamente el de esta selección; **no equivale a una certificación CIS ni a una auditoría integral de todos los controles de un CIS Benchmark**. Los controles se alinean con principios de endurecimiento recomendados; las referencias exactas a IDs y versiones de CIS requieren validación específica por plataforma.

## 2. Objetivos

1. Levantar una topología con router, firewall, switch, servidor Ubuntu y estación Windows.
2. Auditar parámetros de seguridad sobre Cisco IOS y FortiOS.
3. Reducir servicios administrativos innecesarios, usar protocolos seguros y establecer tiempos de inactividad.
4. Registrar resultados **antes y después** del hardening.
5. Verificar acceso administrativo y conectividad sin perder la operatividad de la red.
6. Publicar configuraciones, scripts, reportes y evidencias del laboratorio.

## 3. Topología GNS3

El recorrido físico/lógico del laboratorio es:

```text
                   NAT1 (GNS3 NAT)
                         |
                      nat0
                         |
                    Fa0/1 DHCP
                  +-------------+
                  | R-CIS       |
                  | Cisco 2691  |
                  +-------------+
                    Fa0/0 .1/30
                         |
                    port1 .2/30
                +------------------+
                | FortiGate 7.0.9  |
                +------------------+
                    port2 .129/25
                         |
                       Gi0/0
                   +----------+
                   | SW-CIS   |
                   +----------+
                    /        \
                 Gi0/2      Gi0/1
                  /            \
         Ubuntu Server       Windows 10
        10.21.40.138/25    10.21.40.140/25
```

- [Diagrama de red](Topologia/Diagrama.png)
- [Captura de la topología real en GNS3](Topologia/Topologia.png)

### Direccionamiento

| Nodo | Interfaz | Dirección IP | Puerta de enlace / siguiente salto |
|---|---|---|---|
| GNS3 NAT | `nat0` | Red `192.168.42.0/24` | — |
| Router R-CIS | `Fa0/1` | DHCP; observada `192.168.42.214/24` | Red NAT de GNS3 |
| Router R-CIS | `Fa0/0` | `10.21.40.1/30` | — |
| FortiGate | `port1` | `10.21.40.2/30` | Ruta por defecto vía `10.21.40.1` |
| FortiGate | `port2` | `10.21.40.129/25` | — |
| Switch SW-CIS | `Vlan1` | `10.21.40.130/25` | `10.21.40.129` |
| Ubuntu Server | `ens3` | `10.21.40.138/25` | `10.21.40.129` |
| Windows 10 QEMU | Ethernet | `10.21.40.140/25` | `10.21.40.129` |

**Redes:** enlace router–firewall `10.21.40.0/30`; LAN `10.21.40.128/25`. La IP de la WAN del router puede variar porque se obtiene por DHCP.

Ver [Direccionamiento detallado](Direccionamiento/README.md).

## 4. Herramientas y versiones empleadas

| Herramienta | Uso |
|---|---|
| GNS3 | Emulación de la topología |
| Cisco 2691 — IOS 12.4(25d) | Router R-CIS |
| Cisco IOSvL2 — IOS 15.2 | Switch SW-CIS |
| FortiGate VM — FortiOS 7.0.9 | Firewall |
| Ubuntu Server 24.04.5 LTS | Servidor de automatización |
| Python 3.12.3 | Scripts de auditoría y hardening |
| Paramiko 3.5.1 | Sesiones SSH desde Python |
| Windows 10 QEMU | Equipo de pruebas |

## 5. Controles de hardening evaluados

### Router Cisco R-CIS — 3 controles

| Control | Objetivo | Configuración final |
|---|---|---|
| SSHv2 | Administrar mediante SSH versión 2 | `ip ssh version 2` |
| HTTP | Deshabilitar servidor web no cifrado | `no ip http server` |
| Timeout VTY | Terminar sesiones inactivas a los 5 minutos | `exec-timeout 5 0` |

También se verificaron `login local` y `transport input ssh` en las líneas VTY. El control HTTP **ya cumplía** antes de aplicar cambios.

### Switch Cisco SW-CIS — 3 controles

| Control | Objetivo | Configuración final |
|---|---|---|
| SSHv2 | Utilizar SSH versión 2 | `ip ssh version 2` |
| HTTP | Deshabilitar servidor HTTP | `no ip http server` |
| Timeout VTY | Limitar inactividad a cinco minutos | `exec-timeout 5 0` |

El switch conserva `ip http secure-server` (HTTPS); la deshabilitación evaluada corresponde a **HTTP**, no a HTTPS.

### FortiGate 7.0.9 — 5 controles

| Control | Configuración o estado final | Resultado |
|---|---|---|
| Acceso administrativo WAN | `port1` con `allowaccess ping` solamente | **Cumple** |
| HTTP administrativo LAN | `port2` conserva `ping https ssh http` | **Pendiente** |
| Timeout de administración | `admintimeout 5` | **Cumple** |
| SSH versión 1 | `admin-ssh-v1 disable` | **Cumple** |
| Política de contraseñas | Habilitada para administradores; mínimo 12 caracteres, mayúscula, minúscula, número y símbolo | **Cumple** |

**Excepción importante:** Se intentó utilizar HTTPS administrativo en la LAN, pero se observó un problema de negociación TLS. Por precaución, no se deshabilitó HTTP en `port2` hasta poder confirmar un acceso HTTPS funcional. Este control **no se declara cumplido**.

Más detalles y justificaciones en [Recomendaciones CIS](Documentacion/Recomendaciones_CIS.md).

## 6. Resultados de auditoría: antes y después

Los archivos JSON originales registran lo siguiente:

| Dispositivo | Inicial | Final | Variación |
|---|---:|---:|---:|
| Router R-CIS | **1/3** (33,33 %) | **3/3** (100 %) | +2 controles |
| Switch SW-CIS | **0/3** (0 %) | **3/3** (100 %) | +3 controles |
| FortiGate | **2/5** (40 %) | **4/5** (80 %) | +2 controles |
| **Total** | **3/11** (27,3 %) | **10/11** (90,9 %) | **+7 controles** |

### Reportes originales

| Equipo | Auditoría inicial | Auditoría final |
|---|---|---|
| Router | [router_antes.json](Reportes/router_antes.json) | [router_despues.json](Reportes/router_despues.json) |
| Switch | [switch_antes.json](Reportes/switch_antes.json) | [switch_despues.json](Reportes/switch_despues.json) |
| FortiGate | [fortigate_antes.json](Reportes/fortigate_antes.json) | [fortigate_despues.json](Reportes/fortigate_despues.json) |

También se conserva una [medición intermedia del FortiGate (3/5)](Reportes/fortigate_antes_original.json), diferente del estado inicial y final.

**Nota metodológica:** Las auditorías históricas anteriores al hardening son distintas de las ejecuciones posteriores de `--audit` o `--apply` sobre los dispositivos ya endurecidos. En una ejecución de mantenimiento, `--apply` sobre el switch indicó **«No hay cambios pendientes»** y mantuvo 3/3; esto demuestra el comportamiento sin cambios, no una remediación nueva.

## 7. Scripts de auditoría y hardening

Los archivos fuente documentados están en [Scripts](Scripts/):

- [hardening_router.py.txt](Scripts/hardening_router.py.txt)
- [hardening_switch.py.txt](Scripts/hardening_switch.py.txt)
- [hardening_fortigate.py.txt](Scripts/hardening_fortigate.py.txt)
- [Registro de preparación del servidor Ubuntu](Scripts/scripts-servidor-ubuntu-cis-hardening.txt)

### Modos de uso previstos

```bash
# En Ubuntu, dentro del proyecto y con el entorno virtual activo:
python3 hardening_router.py --audit
python3 hardening_router.py --apply

python3 hardening_switch.py --audit
python3 hardening_switch.py --apply

python3 hardening_fortigate.py --audit
python3 hardening_fortigate.py --apply
```

> **Importante para reproducción:** Los scripts publicados se conservaron con extensión `.py.txt` como material documental. Los comandos anteriores hacen referencia a los archivos originales `.py` que se ejecutaron en Ubuntu. Antes de ejecutarlos en otro entorno, recuperar una copia Python válida, retirar cualquier línea de título añadida al archivo de texto y verificar dependencias, sintaxis y credenciales. **`--apply` puede modificar configuraciones de red**; utilizarlo solamente en un entorno autorizado, revisando el código y disponiendo de acceso por consola. En FortiGate, la secuencia de activación de la política de contraseñas requirió una corrección manual durante la práctica.

No se publican contraseñas operativas como parte de las instrucciones de ejecución.

## 8. Evidencias visuales

Las capturas originales de la práctica están consolidadas en un único documento PDF, con imágenes y breves explicaciones:

**[Ver informe completo de evidencias CIS Hardening (PDF)](Evidencias/Evidencias_CIS_Hardening.pdf)**

El documento incluye topología, auditorías iniciales y finales, controles de router/switch, configuración GUI del FortiGate, conectividad SSH, comprobación HTTP y ejecuciones de `--audit` / `--apply`.

**[Ver documentación gráfica de configuración FortiGate (PDF)](Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf)**

> Las evidencias del modo `--apply` capturadas tras el hardening muestran dispositivos que ya cumplían, por lo que no deben confundirse con capturas de una remediación efectuada en esa misma ejecución.

## 9. Configuraciones finales

- [Router Cisco R-CIS](Configuraciones/Show%20runnig-config%20R-CIS..txt)
- [Switch Cisco SW-CIS](Configuraciones/Show%20runnig-config%20SW-CIS..txt)
- [FortiGate — configuración exportada](Configuraciones/Configuracion%20Fortigate.txt)

**Advertencia de seguridad:** Una exportación completa de FortiGate puede incluir valores de contraseña cifrados `ENC`. Deben ocultarse antes de compartir la configuración públicamente y revisarse los secretos expuestos en el historial del repositorio. Los archivos documentales depurados no deben usarse sin revisión como respaldo de restauración.

## 10. Organización del repositorio

```text
Lab-CIS-Hardening/
├── README.md                         ← Presentación principal
├── Configuracion FortiGate GUI/
│   └── Configuracion_FortiGate_GUI.pdf
├── Configuraciones/
│   ├── Configuracion Fortigate.txt
│   ├── Show runnig-config R-CIS..txt
│   └── Show runnig-config SW-CIS..txt
├── Direccionamiento/
│   └── README.md
├── Documentacion/
│   └── Recomendaciones_CIS.md
├── Evidencias/
│   ├── README.md
│   └── Evidencias_CIS_Hardening.pdf
├── Reportes/
│   ├── router_antes.json
│   ├── router_despues.json
│   ├── switch_antes.json
│   ├── switch_despues.json
│   ├── fortigate_antes.json
│   ├── fortigate_antes_original.json
│   └── fortigate_despues.json
├── Scripts/
│   ├── hardening_router.py.txt
│   ├── hardening_switch.py.txt
│   ├── hardening_fortigate.py.txt
│   └── scripts-servidor-ubuntu-cis-hardening.txt
└── Topologia/
    ├── Diagrama.png
    └── Topologia.png
```

## 11. Pendientes y consideraciones finales

- **Publicar el video de YouTube** y reemplazar la indicación al principio de este README por su enlace auténtico.
- **Completar el TXT de entrega** con los enlaces al repositorio y al video.
- **Pendiente técnico:** resolver HTTPS administrativo en el FortiGate, deshabilitar HTTP en la LAN y comprobar otra vez el control.
- Para reproducibilidad, revisar la publicación de scripts directamente como `.py` y un archivo `requirements.txt`.
- Antes de continuar compartiendo configuraciones, revisar credenciales, hashes y valores cifrados en archivos e historial de Git.

## 12. Referencias

- [CIS Benchmarks — Center for Internet Security](https://www.cisecurity.org/cis-benchmarks)
- [Fortinet FortiOS — documentación de administración](https://docs.fortinet.com/)
- [Recomendaciones CIS y metodología utilizada en este laboratorio](Documentacion/Recomendaciones_CIS.md)

---

**Proyecto académico de seguridad de redes — GNS3 / Cisco IOS / FortiOS / Python.**

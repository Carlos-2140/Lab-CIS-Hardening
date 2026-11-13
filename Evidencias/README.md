# Evidencias — CIS Hardening en GNS3

Esta carpeta está destinada a **capturas auténticas** del laboratorio y a su explicación. No se usan capturas recreadas ni resultados simulados.

## Alcance y resultados

Se auditaron **11 controles seleccionados**, no la totalidad de un CIS Benchmark:

| Equipo | Antes | Después | Reportes JSON |
|---|---:|---:|---|
| Router R-CIS | 1/3 | 3/3 | [Inicial](../Reportes/router_antes.json) · [Final](../Reportes/router_despues.json) |
| Switch SW-CIS | 0/3 | 3/3 | [Inicial](../Reportes/switch_antes.json) · [Final](../Reportes/switch_despues.json) |
| FortiGate 7.0.9 | 2/5 | 4/5 | [Inicial](../Reportes/fortigate_antes.json) · [Final](../Reportes/fortigate_despues.json) |
| **Total** | **3/11** | **10/11** | |

**Excepción documentada:** el acceso administrativo HTTP permanece habilitado en `port2` del FortiGate porque la negociación HTTPS falló. No debe presentarse como control corregido.

## Evidencias visuales requeridas

Las capturas indicadas a continuación deben mostrar claramente el dispositivo o la terminal, el comando o pantalla pertinente y el resultado. Usar la tecla `Win + Shift + S` en Windows QEMU o captura del visor GNS3. Ocultar credenciales, hashes, claves y tokens.

| Código | Nombre sugerido | Qué debe demostrar | Fuente |
|---|---|---|---|
| EV-01 | `EV-01-topologia-gns3.png` | NAT1 → R-CIS → FortiGate → SW-CIS → Ubuntu / Windows | Captura de GNS3; también disponible en [Topologia](../Topologia/) |
| EV-02 | `EV-02-router-auditoria-antes.png` | Resultado de `router_antes.json`: 1/3 | [JSON inicial](../Reportes/router_antes.json) |
| EV-03 | `EV-03-router-auditoria-despues.png` | Resultado de `router_despues.json`: 3/3 | [JSON final](../Reportes/router_despues.json) |
| EV-04 | `EV-04-router-configuracion-ssh-http-vty.png` | `ip ssh version 2`, `no ip http server`, `exec-timeout 5 0` | [Config. router](../Configuraciones/Show%20runnig-config%20R-CIS..txt) |
| EV-05 | `EV-05-switch-auditoria-antes.png` | Resultado de `switch_antes.json`: 0/3 | [JSON inicial](../Reportes/switch_antes.json) |
| EV-06 | `EV-06-switch-auditoria-despues.png` | Resultado de `switch_despues.json`: 3/3 | [JSON final](../Reportes/switch_despues.json) |
| EV-07 | `EV-07-switch-configuracion-ssh-http-vty.png` | `ip ssh version 2`, `no ip http server`, `exec-timeout 5 0` | [Config. switch](../Configuraciones/Show%20runnig-config%20SW-CIS..txt) |
| EV-08 | `EV-08-fortigate-auditoria-antes.png` | Resultado inicial 2/5 | [JSON inicial](../Reportes/fortigate_antes.json) |
| EV-09 | `EV-09-fortigate-auditoria-despues.png` | Resultado final 4/5 y control HTTP LAN pendiente | [JSON final](../Reportes/fortigate_despues.json) |
| EV-10 | `EV-10-fortigate-wan-port1.png` | `port1` 10.21.40.2/30, administración restringida a PING | [GUI documentada](../Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf) |
| EV-11 | `EV-11-fortigate-lan-port2.png` | `port2` 10.21.40.129/25 y accesos conservados | [GUI documentada](../Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf) |
| EV-12 | `EV-12-fortigate-password-policy.png` | Política para Admin, longitud 12 y complejidad | [GUI documentada](../Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf) |
| EV-13 | `EV-13-fortigate-timeout.png` | Timeout de administración de 5 minutos | [GUI documentada](../Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf) |
| EV-14 | `EV-14-fortigate-ruta-y-politica.png` | Ruta por defecto y política LAN-A-INTERNET; pueden ser dos capturas separadas | [GUI documentada](../Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf) |
| EV-15 | `EV-15-prueba-conectividad-y-ssh.png` | Ping y pruebas de acceso TCP/22 desde Ubuntu/Windows según las pruebas reales | Prueba operacional |

## Evidencias que ya se han documentado

- [Capturas de la topología GNS3](../Topologia/).
- [Documento PDF con configuración GUI auténtica del FortiGate](../Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf).
- [Siete reportes JSON de auditoría en la carpeta Reportes](../Reportes/).
- [Running-config del router, switch y configuración FortiGate](../Configuraciones/).

**Importante:** los archivos `EV-XX.png` de la tabla son nombres propuestos, **no imágenes ya subidas**. Solo se debe marcar una evidencia visual como completada después de colocar el PNG auténtico en esta carpeta.

## Cómo obtener las capturas pendientes de auditoría

En Ubuntu Server, ubicado en `~/CIS-Hardening-GNS3`, se pueden **mostrar los reportes existentes sin volver a ejecutar `--apply`**, preservando la situación final del laboratorio:

```bash
cd ~/CIS-Hardening-GNS3
python3 -m json.tool reportes/router_antes.json
python3 -m json.tool reportes/router_despues.json
python3 -m json.tool reportes/switch_antes.json
python3 -m json.tool reportes/switch_despues.json
python3 -m json.tool reportes/fortigate_antes.json
python3 -m json.tool reportes/fortigate_despues.json
```

Tomar una captura por reporte con nombre del archivo y valores legibles. Una captura del JSON **documenta el reporte guardado**, no una nueva ejecución de la auditoría; no etiquetarla como ejecución en vivo.

## Presentación de cada evidencia

Usar este formato en un informe o pie de imagen:

**EV-XX | Nombre descriptivo**

*Figura EV-XX.* Explicación breve de lo que se observa, nombre del dispositivo e indicación del control de hardening verificado.

**Fuente:** captura auténtica del laboratorio GNS3, 13/11/2026, o fecha de captura real si es diferente.

**Resultado:** CUMPLE / NO CUMPLE / PENDIENTE según el reporte correspondiente.

## Relación con documentación técnica

Consultar [Recomendaciones CIS](../Documentacion/Recomendaciones_CIS.md), [Scripts](../Scripts/), [Direccionamiento](../Direccionamiento/) y [Configuraciones](../Configuraciones/).

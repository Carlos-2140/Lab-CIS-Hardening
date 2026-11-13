# Documentación de recomendaciones CIS — Laboratorio GNS3

**Proyecto:** CIS Hardening y auditoría de configuración de dispositivos de red  
**Fecha de evidencias:** 13 de noviembre de 2026  
**Plataforma:** GNS3 — Router Cisco R-CIS (IOS 12.4), Switch Cisco SW-CIS (IOSvL2 15.2), FortiGate (FortiOS 7.0.9)  
**Servidor auditor:** Ubuntu Server 24.04.5 LTS, Python 3.12 y Paramiko 3.5.1

> **Alcance:** Se evaluaron **11 controles seleccionados** para una práctica académica. Los resultados no representan una certificación CIS, una auditoría integral ni el cumplimiento total de un benchmark. El mapeo a recomendaciones se expresa por el objetivo de seguridad; no se atribuyen IDs específicos del benchmark cuando no han sido validados contra la edición exacta utilizada.

## 1. Propósito

Reducir la superficie de ataque de la administración de un router, un switch y un firewall de laboratorio, comprobando el estado de controles antes y después de aplicar cambios. Los scripts admiten `--audit` (consultar sin modificar la configuración) y `--apply` (intentar remediar y volver a auditar). La configuración real, las evidencias y los reportes se conservaron en el repositorio.

El objetivo académico es demostrar **el riesgo**, **la configuración aplicada**, **la comprobación** y **las excepciones justificadas** de cada control.

## 2. Topología y direccionamiento

| Equipo | IP de administración / tránsito | Función |
|---|---|---|
| R-CIS, FastEthernet0/0 | `10.21.40.1/30` | Enlace al FortiGate |
| R-CIS, FastEthernet0/1 | DHCP (`192.168.42.214/24` observado) | Acceso al NAT de GNS3 |
| FortiGate, port1 | `10.21.40.2/30` | WAN hacia el router |
| FortiGate, port2 | `10.21.40.129/25` | Puerta de enlace de la LAN |
| SW-CIS, Vlan1 | `10.21.40.130/25` | Gestión del switch |
| Ubuntu Server, ens3 | `10.21.40.138/25` | Automatización y auditoría |
| Windows 10 QEMU | `10.21.40.140/25` | Pruebas de conectividad |

Ver [documentación de direccionamiento](../Direccionamiento/README.md) y [diagramas del laboratorio](../Topologia/).

## 3. Resultado general

| Equipo | Controles iniciales | Controles finales | Resultado |
|---|---:|---:|---|
| Router R-CIS | 1/3 | 3/3 | 3 controles cumplidos |
| Switch SW-CIS | 0/3 | 3/3 | 3 controles cumplidos |
| FortiGate | 2/5 | 4/5 | 1 control pendiente |
| **Total** | **3/11 (27,3 %)** | **10/11 (90,9 %)** | **7 cumplimientos adicionales** |

Los porcentajes se calculan **solo con los 11 controles de esta práctica**.

## 4. Recomendaciones aplicadas al router Cisco R-CIS

### R-CIS-01 — Forzar SSH versión 2
- **Riesgo:** la administración con protocolos criptográficos obsoletos puede exponer las sesiones a ataques conocidos.
- **Objetivo:** permitir únicamente SSH versión 2.
- **Configuración verificada:** `ip ssh version 2`. En las líneas VTY se configuró `transport input ssh` y `login local`.
- **Comprobación:** `show ip ssh`, `show running-config | include ip ssh version` y revisión de `line vty 0 4`.
- **Resultado:** **NO CUMPLE → CUMPLE**.

### R-CIS-02 — Deshabilitar el servidor HTTP
- **Riesgo:** HTTP administrativo transmite información sin cifrado y aumenta la superficie de exposición.
- **Objetivo:** deshabilitar el servidor de gestión HTTP.
- **Configuración verificada:** `no ip http server`. También se observó `no ip http secure-server` en el router.
- **Comprobación:** `show running-config | include ip http`.
- **Resultado:** **CUMPLE → CUMPLE**. Ya estaba conforme en la auditoría inicial; no debe atribuirse su cumplimiento a un cambio nuevo.

### R-CIS-03 — Limitar inactividad de sesiones VTY
- **Riesgo:** una sesión administrativa abandonada puede ser reutilizada.
- **Objetivo:** cerrar las sesiones VTY inactivas después de 5 minutos.
- **Configuración verificada:** `line vty 0 4` → `exec-timeout 5 0`.
- **Comprobación:** `show running-config | section line vty`.
- **Resultado:** **NO CUMPLE → CUMPLE**.

**Reportes:** [router_antes.json](../Reportes/router_antes.json) · [router_despues.json](../Reportes/router_despues.json).  
**Código:** [hardening_router.py.txt](../Scripts/hardening_router.py.txt).  
**Configuración:** [running-config R-CIS](../Configuraciones/Show%20runnig-config%20R-CIS..txt).

## 5. Recomendaciones aplicadas al switch Cisco SW-CIS

### SW-CIS-01 — Forzar SSH versión 2
- **Riesgo y objetivo:** evitar versiones antiguas del protocolo de administración.
- **Configuración verificada:** `ip ssh version 2`; VTY con `login local` y `transport input ssh`.
- **Comprobación:** `show ip ssh` y revisión de `line vty 0 4`.
- **Resultado:** **NO CUMPLE → CUMPLE**.

### SW-CIS-02 — Deshabilitar HTTP administrativo
- **Riesgo:** gestión sin cifrar por HTTP.
- **Configuración verificada:** `no ip http server`.
- **Comprobación:** `show running-config | include ip http`.
- **Resultado:** **NO CUMPLE → CUMPLE**.
- **Observación:** `ip http secure-server` (HTTPS) permanece configurado; **deshabilitar HTTP no significa deshabilitar HTTPS**.

### SW-CIS-03 — Limitar inactividad de sesiones VTY
- **Riesgo y objetivo:** evitar sesiones administrativas desatendidas.
- **Configuración verificada:** `line vty 0 4` → `exec-timeout 5 0`.
- **Comprobación:** `show running-config | section line vty`.
- **Resultado:** **NO CUMPLE → CUMPLE**.

**Reportes:** [switch_antes.json](../Reportes/switch_antes.json) · [switch_despues.json](../Reportes/switch_despues.json).  
**Código:** [hardening_switch.py.txt](../Scripts/hardening_switch.py.txt).  
**Configuración:** [running-config SW-CIS](../Configuraciones/Show%20runnig-config%20SW-CIS..txt).

## 6. Recomendaciones aplicadas al FortiGate (FortiOS 7.0.9)

### FGT-01 — Restringir acceso administrativo por WAN
- **Riesgo:** exponer HTTP, HTTPS o SSH en una interfaz WAN aumenta la posibilidad de ataques de autenticación y explotación.
- **Objetivo:** no permitir administración remota mediante la interfaz `port1` del laboratorio.
- **Estado inicial:** `set allowaccess ping https ssh http`.
- **Configuración final verificada:** `config system interface` → `edit "port1"` → `set allowaccess ping`.
- **Comprobación:** `show system interface port1` o **Network → Interfaces → port1 → Edit**.
- **Resultado:** **NO CUMPLE → CUMPLE**.

### FGT-02 — Eliminar HTTP administrativo de la LAN
- **Riesgo:** las credenciales y la sesión de administración HTTP no están protegidas mediante TLS.
- **Objetivo:** eliminar `http` de `allowaccess` en `port2`, conservando solo servicios necesarios.
- **Estado inicial y final:** `set allowaccess ping https ssh http`.
- **Motivo de la excepción:** se observó fallo de negociación TLS al intentar acceder por HTTPS al FortiGate. Deshabilitar HTTP antes de recuperar HTTPS podía eliminar el acceso web operativo al dispositivo.
- **Comprobación:** `show system interface port2`, pruebas de acceso web y seguimiento de la incidencia TLS.
- **Resultado:** **NO CUMPLE → NO CUMPLE (PENDIENTE)**.
- **Plan de remediación:** revisar reloj/NTP, certificado y servicio HTTPS; probar exitosamente HTTPS desde la LAN; solo después deshabilitar HTTP, validar la nueva sesión y volver a auditar. **No debe marcarse cumplido sin una prueba real de HTTPS.**

### FGT-03 — Tiempo de inactividad administrativo de cinco minutos
- **Riesgo:** sesiones web o CLI abiertas sin atención.
- **Objetivo:** `admintimeout` no mayor de 5 minutos.
- **Configuración observada:** `set admintimeout 5` (confirmado por auditoría/GUI).
- **Comprobación:** `get system global` o **System → Settings → Idle timeout**.
- **Resultado:** **CUMPLE → CUMPLE**. Ya estaba conforme en el registro inicial.

### FGT-04 — Deshabilitar compatibilidad con SSH versión 1
- **Riesgo:** compatibilidad con una versión criptográficamente obsoleta.
- **Objetivo:** `admin-ssh-v1 disable`.
- **Configuración observada:** `set admin-ssh-v1 disable`.
- **Comprobación:** `get system global` o lectura del valor global mediante CLI.
- **Resultado:** **CUMPLE → CUMPLE**. Ya cumplía antes de la aplicación.

### FGT-05 — Política de contraseñas robustas para administradores
- **Riesgo:** credenciales débiles, más susceptibles a ataques de adivinación.
- **Objetivo:** política activa para administradores, 12 caracteres como mínimo, y al menos una mayúscula, una minúscula, un número y un símbolo.
- **Configuración final observada:**
  ```text
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
- **Comprobación:** `show full-configuration system password-policy`, `get system password-policy` cuando sea compatible, o **System → Settings → Password Policy**.
- **Resultado:** **NO CUMPLE → CUMPLE**.
- **Alcance adicional:** la interfaz gráfica mostró reutilización de contraseñas habilitada y caducidad desactivada; **esos elementos no se incluyeron entre los cinco controles evaluados**.

**Reportes:** [fortigate_antes.json](../Reportes/fortigate_antes.json) · [fortigate_antes_original.json (medición intermedia)](../Reportes/fortigate_antes_original.json) · [fortigate_despues.json](../Reportes/fortigate_despues.json).  
**Código:** [hardening_fortigate.py.txt](../Scripts/hardening_fortigate.py.txt).  
**Configuración:** [FortiGate show](../Configuraciones/Configuracion%20Fortigate.txt).  
**Capturas GUI:** [documentación gráfica del FortiGate](../Configuracion%20FortiGate%20GUI/Configuracion_FortiGate_GUI.pdf).

## 7. Método de prueba y evidencia

1. **Auditoría inicial (`--audit`):** consultar parámetros sin cambios; generar los archivos `*_antes.json`.
2. **Remediación (`--apply`):** aplicar solo los controles seleccionados que faltaban, cuando el acceso y la conectividad lo permitieron.
3. **Nueva comprobación:** generar los archivos `*_despues.json` y verificar `show running-config` / `show` en los dispositivos.
4. **Pruebas operativas:** conectividad ICMP entre nodos y verificación de accesibilidad del servicio SSH/TCP 22; conservación de la ruta y política de salida a Internet.
5. **Excepción documentada:** control HTTP en LAN del FortiGate pendiente, porque HTTPS no estuvo operativo.

**Precisión metodológica:** los scripts y reportes reflejan las pruebas del entorno. Los reportes prueban los **estados observados**, no que todas las modificaciones se hayan realizado de forma autónoma y sin errores. Durante la práctica fue necesaria una corrección manual de la secuencia de la política de contraseñas del FortiGate. No se debe afirmar que el modo `--apply` estuvo libre de incidencias.

### Evidencias enlazadas

- [Reportes JSON finales](../Reportes/)
- [Scripts de auditoría y aplicación](../Scripts/)
- [Configuraciones de dispositivos](../Configuraciones/)
- [Topología real y diagrama](../Topologia/)
- [Direccionamiento IP](../Direccionamiento/)
- [Capturas y configuración gráfica del FortiGate](../Configuracion%20FortiGate%20GUI/)

## 8. Limitaciones y mejoras futuras

- **HTTPS en FortiGate:** resolver la negociación TLS y eliminar `http` del acceso administrativo por LAN.
- **FortiGate:** revisar hosts confiables para administradores, MFA y deshabilitar la reutilización de contraseñas según política institucional.
- **Cisco IOS:** revisar otros controles no incluidos, como AAA, logging centralizado, NTP, banners, cifrado de claves y restricciones de origen para SSH.
- **Evidencia y seguridad:** nunca publicar contraseñas, hashes, valores `ENC`, claves ni tokens de configuraciones.
- **Reproducibilidad:** revisar y probar los archivos Python antes de considerarlos código ejecutable de distribución, y mantener un `requirements.txt` con dependencias.
- **Alcance normativo:** verificar la edición de benchmark apropiada para Cisco IOS 12.4, IOSvL2 15.2 y FortiOS 7.0.9 antes de atribuir códigos o niveles de perfil específicos.

## 9. Fuentes técnicas y referencias

- Center for Internet Security, [CIS Benchmarks (catálogo oficial)](https://www.cisecurity.org/cis-benchmarks). Consultar la edición específica de cada producto.
- Center for Internet Security, [Cisco IOS Benchmark v2.2 (documento oficial histórico)](https://www.cisecurity.org/-/jssmedia/Project/cisecurity/cisecurity/data/media/files/uploads/2017/04/CIS_Cisco_IOS_Benchmark_v22.pdf). Referencia de principios como deshabilitar servicios HTTP innecesarios.
- Fortinet, [FortiOS 7.0 Best Practices — Administrative settings](https://docs.fortinet.com/document/fortigate/7.0.0/best-practices/103945/administrative-settings).
- Fortinet, [FortiOS 7.0 Best Practices (PDF)](https://fortinetweb.s3.amazonaws.com/docs.fortinet.com/v2/attachments/7c8e21ef-1be2-11ec-8c53-00505692583a/FortiOS-7.0.0-Best_Practices.pdf).
- Fortinet, [FortiOS 7.0 CLI Reference — system global](https://docs.fortinet.com/document/fortigate/7.0.1/cli-reference/001620/config-system-global).

**Nota de uso:** Las recomendaciones de hardening citadas se presentan como referencias de seguridad y buenas prácticas. Esta práctica únicamente verificó los 11 controles enumerados y los resultados documentados en sus archivos JSON.

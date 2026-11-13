# Reportes de auditoría — CIS Hardening GNS3

Esta carpeta recopila los **reportes JSON auténticos generados por los scripts** de auditoría y hardening del laboratorio. No contiene resultados simulados.

## Archivos principales que se deben cargar

| Equipo | Auditoría inicial | Auditoría final |
|---|---|---|
| Router Cisco R-CIS | `router_antes.json` | `router_despues.json` |
| Switch Cisco SW-CIS | `switch_antes.json` | `switch_despues.json` |
| FortiGate 7.0.9 | `fortigate_antes.json` | `fortigate_despues.json` |

Los resultados observados en el laboratorio fueron:

- **Router:** 1/3 controles al inicio → 3/3 al final.
- **Switch:** 0/3 controles al inicio → 3/3 al final.
- **FortiGate:** 2/5 controles al inicio → 4/5 al final.

Estos son **controles seleccionados** para la práctica; no representan una certificación ni una evaluación completa de todos los requisitos CIS. En FortiGate quedó pendiente el control de deshabilitar HTTP administrativo en la LAN debido al fallo de acceso HTTPS.

## Reportes adicionales disponibles en Ubuntu

En la carpeta `~/CIS-Hardening-GNS3/reportes/` se generaron, para cada equipo, cuatro variantes que se pueden conservar para trazabilidad:

- `<equipo>_antes_original.json`
- `<equipo>_antes.json`
- `<equipo>_pre_aplicacion.json`
- `<equipo>_despues.json`

Sustituir `<equipo>` por `router`, `switch` o `fortigate`. Se pueden subir los **12 reportes** para conservar la secuencia de pruebas, o primero los **6 principales** de la tabla.

## Cómo subir los archivos

1. En Ubuntu, comprobar los reportes originales con `ls -lh ~/CIS-Hardening-GNS3/reportes/`.
2. Trasladar los JSON a Windows físico con el método de transferencia disponible en GNS3.
3. Abrir la carpeta **Reportes** de este repositorio y seleccionar **Add file → Upload files**.
4. Subir los archivos JSON originales, sin alterar sus resultados, y confirmar el cambio.

**Antes de publicar:** comprobar que los JSON no contengan contraseñas, tokens, claves ni otros secretos.

> Este README organiza la carpeta. **Los JSON deben añadirse a partir de los reportes reales guardados en el servidor Ubuntu; no se han generado nuevos resultados en GitHub.**

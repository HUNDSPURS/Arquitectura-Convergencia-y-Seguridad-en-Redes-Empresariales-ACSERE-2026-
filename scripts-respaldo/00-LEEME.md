# Scripts de respaldo de configuraciones (ACSERE-2026)

El enunciado (7.2) pide en el repositorio "scripts de respaldo", y el informe (Figura 19 y Anexo C, carpeta `respaldos-tftp/`) indica que los respaldos se guardan por TFTP en **SRV-FTP-TFTP (172.16.4.73)**.

## Antes de empezar
1. SRV-FTP-TFTP > Services > **TFTP = On**.
2. Desde cualquier equipo, `ping 172.16.4.73` debe responder (si falla, revisar OSPF/HSRP antes de respaldar).

## Cómo usar cada script
Abrir el equipo > CLI > `enable` (clave `class`) y pegar lo que está debajo de la línea `INICIO` del archivo de ese equipo. Cada script:
1. Prueba alcance al servidor (`ping 172.16.4.73`).
2. Guarda la configuración en NVRAM (`write memory`).
3. Sube la running-config como `<HOSTNAME>-confg`.
4. Sube la startup-config como `<HOSTNAME>-startup-confg`.

Si Packet Tracer no toma bien el pegado de varias líneas, escribir el `copy` a mano y responder las preguntas con las líneas que siguen (IP del servidor y nombre de archivo).

| Archivo | Equipo |
|---|---|
| 01 a 03 | R-CENTRAL, R-NORTE, R-SUR |
| 04 y 05 | MLS-CORE1, MLS-CORE2 |
| 06 a 11 | SW-ACC-DIR, FIN, LEGAL, OPS1, OPS2, SIS |
| 12 y 13 | SW-NORTE, SW-SUR |
| 14 | ISP (necesita abrir y cerrar un permiso temporal en ACL-INTERNET-IN de R-CENTRAL, o guardar su `show run` a mano) |
| 15 | Restauración desde TFTP (`copy tftp: running-config`) |
| 16 | Opcional: respaldo de la imagen IOS (`copy flash: tftp:`) |

La topología con cada switch de acceso conectado a ambos MLS y el triángulo WAN entre los tres routers no cambia nada en los scripts: los equipos llegan al servidor por la ruta que elija OSPF/STP.

## Verificación (evidencia para el informe)
- SRV-FTP-TFTP > Services > TFTP: deben aparecer los 28 archivos `*-confg` (14 equipos x 2). Esa es la **Captura 19**.
- En cada equipo, el IOS muestra `[OK - xxxx bytes]` tras cada copia.

## Para la carpeta comprimida
- `scripts-respaldo/`: esta carpeta tal cual.
- `respaldos-tftp/`: Packet Tracer no permite descargar los archivos del servidor TFTP al PC, así que ahí van la Captura 19 y una copia del `show running-config` de cada equipo (el mismo contenido que se subió por TFTP).
- Demostración sugerida en la presentación: en un switch de acceso hacer `copy startup-config tftp:` → borrar con `erase startup-config` + `reload` → reconfigurar la VLAN 99 mínima → restaurar con el script 15.

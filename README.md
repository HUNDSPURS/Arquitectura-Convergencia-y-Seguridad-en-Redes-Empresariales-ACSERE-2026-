# Arquitectura, Convergencia y Seguridad en Redes Empresariales (ACSERE-2026)

Proyecto final del curso **Redes de Computadoras I**. Diseñamos, configuramos, probamos y documentamos en **Cisco Packet Tracer** la red corporativa convergente de una organización multiservicio con una sede Central y dos sucursales (Norte y Sur). La red integra segmentación por VLAN, redundancia en capa 2 y 3, enrutamiento dinámico, servicios de red, Wi‑Fi para invitados y controles de seguridad.

---

## Integrantes

| Nombre | Carné | Rol en el proyecto |
|---|---|---|
| Kevin Esteban Quinteros Quiñonez | 9989-23-17397 | Ingeniero de conmutación (capa 2) |
| Jose Pablo Valdez de León | 9989-23-85 | Ingeniero de servicios y seguridad |
| Dulce María Morales Ramírez | 9989-23-18927 | Ingeniera de enrutamiento y WAN (capa 3) |
| Diego Alejandro Morales Flores | 9989-23-13177 | Líder de proyecto y arquitecto de red |

- **Curso:** Redes de Computadoras I, Grupo II
- **Universidad:** Universidad Mariano Galvez de Guatemala
- **Catedrático:** David Marroquín
- **Fecha de entrega:** 31/10/2026

---

## Topología

> Para que la imagen aparezca, exporta una captura del `.pkt` y súbela como `docs/topologia.png`.

Topología **híbrida jerárquica**:

- **Sede Central:** núcleo colapsado con dos switches multicapa (MLS-CORE1 y MLS-CORE2, Catalyst 3650) unidos por un EtherChannel LACP. Seis switches de acceso 2960 forman una **malla parcial**, porque cada uno se conecta a los dos núcleos. R-CENTRAL (ISR4331) es el router de borde hacia el ISP.
- **WAN:** **triángulo** de enlaces seriales PPP entre R-CENTRAL, R-NORTE y R-SUR, de modo que si cae un enlace el tráfico se va por los otros dos.
- **Sucursales:** R-NORTE y R-SUR hacen Router-on-a-Stick para su VLAN de usuarios y su VLAN de gestión.
- **Internet simulado:** router ISP con una loopback 8.8.8.8.

---

## Direccionamiento

Red base **172.16.0.0/16**. Las LAN usan **VLSM** (de la subred más grande a la más pequeña) y los enlaces punto a punto usan **FLSM /30** dentro de 172.16.255.0/24.

### VLSM por VLAN

| VLAN | Departamento | Sede | Hosts req. | Red | Máscara | Gateway |
|---|---|---|---|---|---|---|
| 60 | Invitados / Móviles (Wi‑Fi) | Central | 250 | 172.16.0.0/24 | 255.255.255.0 | 172.16.0.1 |
| 40 | Operaciones Logísticas | Central | 180 | 172.16.1.0/24 | 255.255.255.0 | 172.16.1.1 |
| 40 | Usuarios Sucursal Norte | Norte | 180 | 172.16.2.0/24 | 255.255.255.0 | 172.16.2.1 |
| 50 | Finanzas y Atención al Cliente | Central | 120 | 172.16.3.0/25 | 255.255.255.128 | 172.16.3.1 |
| 40 | Usuarios Sucursal Sur | Sur | 120 | 172.16.3.128/25 | 255.255.255.128 | 172.16.3.129 |
| 20 | Legal | Central | 45 | 172.16.4.0/26 | 255.255.255.192 | 172.16.4.1 |
| 30 | Sistemas + granja de servidores | Central | 30 | 172.16.4.64/26 | 255.255.255.192 | 172.16.4.65 |
| 10 | Dirección Ejecutiva | Central | 25 | 172.16.4.128/27 | 255.255.255.224 | 172.16.4.129 |
| 99 | Gestión Central | Central | 10 | 172.16.4.160/27 | 255.255.255.224 | 172.16.4.161 |
| 99 | Gestión Norte | Norte | 5 | 172.16.4.192/28 | 255.255.255.240 | 172.16.4.193 |
| 99 | Gestión Sur | Sur | 5 | 172.16.4.208/28 | 255.255.255.240 | 172.16.4.209 |

En la Central, el gateway de cada VLAN es una **IP virtual HSRP** (la primera IP de la subred). MLS-CORE1 usa la siguiente IP con prioridad 110 (activo) y MLS-CORE2 la que sigue con prioridad 100 (respaldo); por ejemplo, en la VLAN 20: virtual .1, CORE1 .2 y CORE2 .3. VLAN nativa **999** y VLAN de puertos no usados **998**.

### FLSM WAN (/30)

| Enlace | Red | Extremo A | Extremo B |
|---|---|---|---|
| Central – Norte | 172.16.255.0/30 | R-CENTRAL Se0/1/0 .1 | R-NORTE Se0/1/0 .2 |
| Central – Sur | 172.16.255.4/30 | R-CENTRAL Se0/1/1 .5 | R-SUR Se0/1/0 .6 |
| Norte – Sur (respaldo) | 172.16.255.8/30 | R-NORTE Se0/1/1 .9 | R-SUR Se0/1/1 .10 |
| Central – ISP | 172.16.255.12/30 | R-CENTRAL Se0/2/0 .13 | ISP Se0/1/0 .14 |
| R-CENTRAL – MLS-CORE1 | 172.16.255.16/30 | R-CENTRAL G0/0/1 .17 | MLS-CORE1 G1/0/24 .18 |
| R-CENTRAL – MLS-CORE2 | 172.16.255.20/30 | R-CENTRAL G0/0/0 .21 | MLS-CORE2 G1/0/24 .22 |

### Servidores (VLAN 30)

| Servidor | IP | Servicio |
|---|---|---|
| SRV-DNS | 172.16.4.70 | DNS (`www.glfs.com`) |
| SRV-DHCP | 172.16.4.71 | DHCP para todas las VLAN |
| SRV-WEB | 172.16.4.72 | Portal web HTTP |
| SRV-FTP-TFTP | 172.16.4.73 | FTP y TFTP (respaldos) |
| SRV-MAIL | 172.16.4.74 | Correo SMTP/POP3 |

---

## Tecnologías implementadas

- **VLANs y troncales 802.1Q**, con VLAN nativa y VLAN de puertos no usados separadas.
- **STP** sobre la malla parcial de acceso.
- **EtherChannel LACP** entre los dos núcleos.
- **HSRP** para gateway redundante en cada VLAN de la Central.
- **Enrutamiento entre VLANs** con SVIs en los 3650 y Router-on-a-Stick en las sucursales.
- **OSPF** proceso 10, área 0.
- **PPP** en los enlaces seriales WAN.
- **DHCP** centralizado con `ip helper-address`.
- **Servicios:** DNS, Web, FTP/TFTP, correo y NTP.
- **Wi‑Fi de invitados:** SSID `GLFS-INVITADOS`, WPA2‑PSK con AES, aislado en la VLAN 60.
- **Seguridad:** SSH v2, ACL estándar en las líneas VTY (solo Sistemas y Gestión administran), Port Security en los puertos de acceso y ACL extendidas (ACL-INVITADOS, ACL-OPERACIONES, ACL-FINANZAS y ACL-INTERNET-IN en el borde).
- **Respaldos de configuración por TFTP.**

---

## Estructura del repositorio

```
.
├── Configuraciones/     Archivos .pkt de Packet Tracer (topología, conexiones,
│                        tablas de enrutamiento y configuración de los equipos)
├── scripts-respaldo/    Un script por equipo para respaldar su configuración
│                        por TFTP, más el script de restauración
└── README.md
```

Contenido de `scripts-respaldo/`:

| Archivo | Equipo |
|---|---|
| `00-LEEME.md` | Instrucciones de uso |
| `01` a `03` | R-CENTRAL, R-NORTE, R-SUR |
| `04` y `05` | MLS-CORE1, MLS-CORE2 |
| `06` a `11` | SW-ACC-DIR, FIN, LEGAL, OPS1, OPS2, SIS |
| `12` y `13` | SW-NORTE, SW-SUR |
| `14` | ISP (requiere un permiso temporal en ACL-INTERNET-IN) |
| `15` | Restauración desde TFTP |
| `16` | Opcional: respaldo de la imagen IOS |

---

## Cómo usarlo

### Requisitos

- **Cisco Packet Tracer 9.0.1 o superior** (versiones anteriores pueden no abrir el archivo).

### Abrir la topología

1. Clona o descarga el repositorio (**Code > Download ZIP**).
2. Abre el `.pkt` de la carpeta `Configuraciones/` con Packet Tracer.
3. Presiona **Fast Forward Time** unas veces para que converjan STP, OSPF y HSRP.
4. Si alguna PC aparece con IP 169.254.x.x, entra a **Desktop > IP Configuration** y vuelve a seleccionar **DHCP**.

### Respaldar una configuración

1. En SRV-FTP-TFTP, verifica que **Services > TFTP** esté en **On**.
2. Abre el equipo > **CLI** > `enable`.
3. Pega el contenido del script de ese equipo (lo que está debajo de la línea `INICIO`).
4. El archivo aparece en el servidor como `<HOSTNAME>-confg`.

### Restaurar desde TFTP

Si un equipo perdió su configuración, primero dale lo mínimo para llegar a 172.16.4.73 (IP de gestión y gateway en un switch, o IP de interfaz y ruta en un router) y después ejecuta:

```
copy tftp: running-config
172.16.4.73
NOMBRE-EQUIPO-confg
running-config
write memory
```

El detalle está en `scripts-respaldo/15-restaurar-desde-tftp.txt`.

---

## Credenciales de laboratorio

| Acceso | Usuario | Contraseña |
|---|---|---|
| Modo privilegiado (`enable`) | — | `class` |
| Consola | — | `cisco` |
| SSH | `admin` | `Admin2026` |
| Wi‑Fi invitados (`GLFS-INVITADOS`) | — | `Invitados2026!` |

> ⚠️ **Advertencia:** estas credenciales son solo para el laboratorio académico en Packet Tracer. No deben reutilizarse en equipos ni redes reales.

---

## Licencia

Proyecto académico con fines educativos. Redes de Computadoras I, Grupo II, 2026.


Práctica 1 - Seguridad perimetral 

Mario Josó Pujols De La Cruz 
2022-1453


--- Video de demostración



--- Objetivo del laboratorio

El objetivo de este laboratorio es implementar una infraestructura de red segura utilizando FortiGate como firewall principal, un switch Cisco y diferentes VLAN para separar usuarios, servidor web, servidor de base de datos y administración.

También se implementaron controles de seguridad como políticas de firewall, NAT, DHCP, Port-Security, SSH, filtrado de archivos, IPS y protección contra ataques DoS.



--- La topología está compuesta por:

- 1 FortiGate.
- 1 switch Cisco.
- 1 red de usuarios.
- 1 servidor WEB.
- 1 servidor de Base de Datos.
- Conexión hacia Internet mediante NAT.

--- Direccionamiento IP

| Segmento | VLAN | Red | Gateway |
|---|---:|---|---|
| Usuarios | 10 | 10.14.53.0/25 | 10.14.53.1 |
| Servidor WEB | 20 | 10.14.53.128/28 | 10.14.53.129 |
| Servidor DB | 30 | 10.14.53.144/28 | 10.14.53.145 |
| Administración | 99 | 10.14.53.160/28 | 10.14.53.161 |

---  Equipos principales

- PC Usuarios: `10.14.53.10`
- Servidor WEB: `10.14.53.130`
- Servidor DB: `10.14.53.146`
- Switch: `10.14.53.162`
- FortiGate VLAN 10: `10.14.53.1`
- FortiGate VLAN 20: `10.14.53.129`
- FortiGate VLAN 30: `10.14.53.145`
- FortiGate VLAN 99: `10.14.53.161`

---  Configuración del FortiGate

En el FortiGate se configuraron subinterfaces VLAN sobre la interfaz conectada al switch.

Las VLAN utilizadas fueron:

- VLAN 10 - Usuarios.
- VLAN 20 - Servidor WEB.
- VLAN 30 - Servidor de Base de Datos.
- VLAN 99 - Administración.

También se configuró DHCP para la red de usuarios y una ruta por defecto hacia Internet.

---  NAT

Se creó una política desde la VLAN de usuarios hacia la interfaz WAN del FortiGate con NAT habilitado.

Esto permite que los equipos de la red interna puedan acceder a Internet utilizando la dirección IP asignada a la interfaz WAN del firewall.

--- Políticas de Firewall

--- Usuarios hacia Internet

Se permite la salida de los usuarios hacia Internet utilizando NAT.

--- Usuarios hacia Servidor WEB

Se permite exclusivamente tráfico HTTPS hacia el servidor:

`10.14.53.130`

Puerto permitido:

`TCP/443`

Esta política también tiene configurados perfiles de IPS y File Filter.

--- Usuarios hacia Base de Datos

Los usuarios tienen bloqueado el acceso directo hacia:

`10.14.53.146`

Puerto:

`TCP/3306 - MySQL`

Durante las pruebas, FortiGate registró los intentos como:

`Deny: policy violation`

--- Servidor WEB hacia Base de Datos

El servidor WEB puede comunicarse con el servidor de Base de Datos exclusivamente utilizando:

`TCP/3306`

La prueba realizada desde el servidor WEB confirmó:

`10.14.53.130 -> 10.14.53.146:3306 = permitido`

También se realizó una prueba hacia TCP/22:

`10.14.53.130 -> 10.14.53.146:22 = bloqueado`

---  Servidor WEB

El servidor WEB utiliza Ubuntu Server con Apache.

Dirección IP:

`10.14.53.130/28`

Gateway:

`10.14.53.129`

Se configuró HTTPS utilizando un certificado RSA de 2048 bits.

El acceso desde la red de usuarios hacia:

`https://10.14.53.130`

funcionó correctamente.

## Servidor de Base de Datos

El servidor de Base de Datos utiliza MariaDB.

Dirección IP:

`10.14.53.146/28`

Gateway:

`10.14.53.145`

Se creó una base de datos de laboratorio para realizar pruebas de conectividad y seguridad.

--- Protección contra DoS

Se configuró una política IPv4 DoS para proteger el servidor WEB.

La anomalía utilizada para la prueba fue:

`tcp_syn_flood`

Se configuró un umbral bajo únicamente con fines de laboratorio.

Durante la prueba, FortiGate detectó múltiples eventos SYN Flood desde:

`10.14.53.10`

hacia:

`10.14.53.130:443`

FortiGate registró los eventos como críticos y aplicó acciones para limpiar las sesiones detectadas.

--- IPS contra SQL Injection

Se creó un perfil IPS para detectar diferentes firmas relacionadas con ataques SQL Injection.

Entre las firmas configuradas se encuentran:

- HTTP URI SQL Injection.
- HTTP Header SQL Injection.
- HTTP Referer SQL Injection.

El perfil fue aplicado a la política entre los usuarios y el servidor WEB.

La configuración quedó implementada, aunque durante las pruebas realizadas no fue posible obtener evidencia definitiva de bloqueo del ataque.

--- File Filter

Se creó un perfil de File Filter llamado:

`bloqueo-exe-web`

El objetivo es bloquear archivos ejecutables `.exe`.

El perfil fue aplicado a la política de acceso al servidor WEB.

La configuración del filtro quedó implementada, aunque durante las pruebas del laboratorio el archivo de prueba no fue bloqueado correctamente por el FortiGate utilizado.

--- Configuración del Switch

En el switch Cisco se configuraron las siguientes VLAN:

- VLAN 10 - Usuarios.
- VLAN 20 - Servidor WEB.
- VLAN 30 - Servidor DB.
- VLAN 99 - Administración.

El enlace hacia el FortiGate fue configurado como trunk permitiendo las VLAN:

`10,20,30,99`

Los puertos utilizados fueron:

- Gi0/0 - Enlace troncal hacia FortiGate.
- Gi0/1 - Usuarios.
- Gi0/2 - Servidor WEB.
- Gi0/3 - Servidor DB.

Los puertos no utilizados fueron asignados a la VLAN 99 y apagados administrativamente.

--- Seguridad del Switch

Se implementaron diferentes medidas de seguridad:

- SSH versión 2.
- Usuario administrador local.
- Contraseñas cifradas.
- Banner de advertencia.
- Port-Security.
- Sticky MAC.
- Máximo de 2 direcciones MAC.
- Shutdown ante violaciones.
- Puertos no utilizados apagados.

--- Pruebas realizadas

Durante el laboratorio se realizaron diferentes pruebas de seguridad y conectividad.

--- HTTPS

Usuarios hacia servidor WEB:

`HTTPS TCP/443 -> Permitido`

--- Usuarios hacia Base de Datos

`TCP/3306 -> Bloqueado`

FortiGate registró:

`Deny: policy violation`

--- Servidor WEB hacia Base de Datos

`TCP/3306 -> Permitido`

--- Servidor WEB hacia SSH del DB

`TCP/22 -> Bloqueado`

--- Ataque SYN Flood

Se generaron múltiples paquetes SYN hacia el servidor WEB.

FortiGate detectó:

`tcp_syn_flood`

y registró el evento con severidad crítica.

--- Resultados

La implementación permitió separar los diferentes servicios utilizando VLAN y aplicar políticas específicas entre cada segmento de red.

Se verificó correctamente:

- Acceso HTTPS al servidor WEB.
- DHCP para usuarios.
- NAT hacia Internet.
- Bloqueo de usuarios hacia el servidor DB.
- Comunicación WEB a DB exclusivamente mediante MySQL.
- Bloqueo de otros servicios entre WEB y DB.
- Protección contra SYN Flood.
- Segmentación mediante VLAN.
- Port-Security.
- SSH.
- Trunk 802.1Q.
- Puertos no utilizados apagados.

Además, fueron configurados perfiles de IPS contra SQL Injection y File Filter para archivos ejecutables.

## Conclusión

Este laboratorio permitió implementar una red segmentada y aplicar diferentes medidas de seguridad utilizando FortiGate y un switch Cisco.

La separación de usuarios, servidores y administración mediante VLAN permite controlar mejor el tráfico entre los diferentes segmentos.

Las políticas del firewall permitieron limitar el acceso únicamente a los servicios necesarios, mientras que controles adicionales como Port-Security, SSH, IPS y protección DoS aumentan la seguridad general de la infraestructura.

Las pruebas realizadas permitieron comprobar que FortiGate puede detectar y bloquear tráfico no autorizado entre diferentes redes y también identificar comportamientos asociados con ataques de denegación de servicio.

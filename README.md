# Informe de Incidente de Seguridad (Fase 1)

## 1. Introducción

Se ha realizado un análisis forense sobre un servidor Debian comprometido, con el objetivo de identificar el origen del ataque, detectar las vulnerabilidades explotadas y aplicar medidas correctivas para restaurar la seguridad del sistema.

---

## 2. Análisis del acceso

Se analizaron los logs del sistema utilizando `journalctl`, debido a la ausencia del archivo `/var/log/auth.log`.

Se detectó un acceso sospechoso mediante SSH:

* Usuario: **root**
* Método: **autenticación por contraseña**
* IP de origen: **192.168.0.13**

Esto indica un acceso no autorizado al sistema.

**Evidencias:**

* [`fase1_1_logs_ssh.png`](./evidencias/fase1/fase1_1_logs_ssh.png)
* `fase1_1_logs_ssh1.png`

---

## 3. Análisis de sesiones

Se utilizó el comando `last` para analizar sesiones activas y pasadas.

No se encontraron registros del acceso root detectado previamente, lo que sugiere:

* posible manipulación de logs
* o eliminación de evidencias por parte del atacante

**Evidencia:**

* `fase1_2_last.png`

---

## 4. Análisis de procesos

Se analizaron los procesos en ejecución mediante `ps aux`.

No se detectaron procesos sospechosos ni malware activo en memoria.

**Evidencia:**

* `fase1_3_procesos.png`

---

## 5. Análisis de puertos y servicios

Se identificaron servicios expuestos mediante `netstat -tulnp`:

* SSH (puerto 22) → accesible desde cualquier IP
* HTTP (puerto 80)
* FTP (puerto 21)

El servicio FTP representa un riesgo de seguridad si no está correctamente configurado.

**Evidencia:**

* `fase1_4_puertos.png`

---

## 6. Detección de malware

Se utilizó la herramienta `rkhunter` para detectar rootkits.

Resultados:

* 1 archivo sospechoso
* 4 posibles rootkits

No se encontraron evidencias concluyentes de compromiso activo, por lo que se consideran posibles falsos positivos.

**Evidencia:**

* `fase1_5_rootkit.png`

---

## 7. Medidas de mitigación

Se aplicaron las siguientes acciones para mitigar el ataque:

### Seguridad en SSH

* Se deshabilitó el acceso root:

  * `PermitRootLogin no`
* Se deshabilitó la autenticación por contraseña:

  * `PasswordAuthentication no`

### Servicios

* Se detuvo y deshabilitó el servicio FTP (`vsftpd`)

### Actualización del sistema

* Se actualizaron paquetes del sistema:

  * `apt update && apt upgrade`

### Gestión de credenciales

* Se cambió la contraseña del usuario

### Firewall

* Se instaló y configuró `ufw`
* Se habilitó el firewall
* Se permitió únicamente el acceso SSH

**Evidencias:**

* `fase1_6_mitigacion.png`
* `fase1_7_password.png`
* `fase1_7_firewall.png`

---

## 8. Recomendaciones

* Utilizar autenticación por clave pública en SSH
* Deshabilitar completamente el acceso root remoto
* Monitorizar logs de forma continua
* Implementar políticas de contraseñas seguras
* Mantener el sistema actualizado regularmente
* Limitar servicios expuestos únicamente a los necesarios
* Configurar herramientas de detección de intrusos

---

## 9. Conclusión

El sistema ha sido comprometido mediante un acceso SSH inseguro. Tras el análisis forense y la aplicación de medidas correctivas, el sistema ha sido asegurado, reduciendo significativamente la superficie de ataque y previniendo accesos no autorizados futuros.

#  Fase 2 - Pentesting y Hardening del Sistema

##  Introducción

En esta fase se ha realizado un proceso de **pentesting controlado** sobre el servidor Debian con servicios web y base de datos activos.  

El objetivo ha sido:

- Identificar servicios expuestos
- Enumerar posibles vectores de ataque
- Explotar vulnerabilidades reales
- Aplicar medidas de mitigación y hardening

---

##  Metodología

Se ha seguido un enfoque estándar de pentesting:

1. Reconocimiento
2. Escaneo de servicios
3. Enumeración
4. Explotación
5. Post-explotación
6. Mitigación

---

##  1. Reconocimiento

Se identificó la dirección IP de la máquina objetivo.

![IP de la máquina](./evidencias/fase2_1_ip_maquina.png)

---

##  2. Escaneo de puertos (Nmap)

Se realizó un escaneo completo de puertos:

```bash
nmap -p- -sS -sV <IP>
```

### Resultados:

- Puerto 22 → SSH
- Puerto 80 → HTTP (WordPress)
- Puerto 3306 → MySQL

![Nmap 1](./evidencias/fase2_2_nmap_completo1.png)
![Nmap 2](./evidencias/fase2_2_nmap_completo2.png)

---

##  3. Enumeración de servicios

###  Acceso a MySQL

Se logró acceso a la base de datos:

```bash
mysql -u user -p
```

![Login MySQL](./evidencias/fase2_3_mysql_login.png)

---

###  Enumeración de bases de datos

```sql
SHOW DATABASES;
```

![Enumeración MySQL](./evidencias/fase2_4_mysql_enum.png)

---

###  Acceso a base de datos WordPress

```sql
USE wordpress;
SHOW TABLES;
```

![Tablas WordPress](./evidencias/fase2_5_wordpress_enum.png)

---

###  Extracción de usuarios

```sql
SELECT ID, user_login, user_pass FROM wp_users;
```

### Resultado:

- Usuario: wordpress-user  
- Hash de contraseña obtenido

![Usuarios WordPress](./evidencias/fase2_6_wordpress_users.png)

---

##  4. Vulnerabilidades detectadas

Se identificaron las siguientes debilidades:

- Acceso a MySQL sin restricciones adecuadas
- Exposición de credenciales de WordPress
- Posibilidad de acceso remoto a la base de datos
- Configuración por defecto insegura

---

##  5. Mitigación y Hardening

###  Cambio de contraseña de MySQL (root)

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'ContraseñaMUYSegura123!';
FLUSH PRIVILEGES;
```

![Cambio contraseña MySQL](./evidencias/fase2_5_mysql_password.png)

---

###  Restricción de acceso remoto a MySQL

Edición del archivo:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Configuración aplicada:

```text
bind-address = 127.0.0.1
```

Esto limita el acceso únicamente a localhost.

---

###  Reinicio del servicio

```bash
sudo systemctl restart mysql
```

---

###  Buenas prácticas aplicadas

- Uso de contraseñas seguras
- Restricción de acceso a servicios críticos
- Eliminación de exposición innecesaria
- Principio de mínimo privilegio

---

##  6. Impacto de las vulnerabilidades

Las vulnerabilidades detectadas podían permitir:

- Acceso no autorizado a la base de datos
- Robo de credenciales de usuarios
- Compromiso total del sistema
- Escalada de privilegios

---

##  7. Conclusión

El sistema presentaba vulnerabilidades críticas derivadas de configuraciones inseguras por defecto.

Tras la fase de explotación y posterior hardening:

- Se ha eliminado el acceso remoto a MySQL
- Se han protegido las credenciales críticas
- Se ha reforzado la seguridad general del sistema

El entorno queda ahora significativamente más seguro frente a ataques externos.

---

## Entregables

### Packet Tracer
Archivo incluido en el repositorio.

###  Máquina Virtual

Formato: OVA (VirtualBox)

🔗 Descargar:
https://drive.google.com/file/d/14leqmWU1Ry_9Zk-Lzh59UB2wln6Gfge3/view?usp=sharing

# Fase 3: Plan de Respuesta a Incidentes y SGSI (ISO 27001)

## 1. Introducción

En esta fase se desarrolla un plan de respuesta a incidentes basado en la guía NIST SP 800-61, junto con la implementación de un Sistema de Gestión de Seguridad de la Información (SGSI) alineado con la norma ISO 27001.

El objetivo es dotar a la organización de procedimientos estructurados que permitan detectar, analizar, contener y recuperar de incidentes de seguridad, así como establecer un marco de mejora continua que reduzca la probabilidad de futuros ataques.

Este plan se basa en el incidente previamente analizado, en el que se detectó un acceso no autorizado al sistema mediante SSH, posiblemente utilizando credenciales comprometidas.

---

## 2. Plan de Respuesta a Incidentes (NIST SP 800-61)

El plan se estructura en seis fases clave:

---

### 2.1 Preparación

La fase de preparación es fundamental para garantizar una respuesta eficaz ante incidentes.

Se han definido las siguientes medidas:

- Establecimiento de roles y responsabilidades dentro del equipo de seguridad.
- Implementación de herramientas de monitorización y análisis de logs.
- Configuración de registros del sistema (logs) para detectar actividad sospechosa.
- Formación básica en seguridad para los usuarios.
- Definición de procedimientos de actuación ante incidentes.
- Realización de copias de seguridad periódicas.

Estas medidas permiten a la organización estar preparada para actuar rápidamente ante cualquier amenaza.

---

### 2.2 Identificación

La identificación consiste en detectar y confirmar la existencia de un incidente de seguridad.

Para ello se utilizan:

- Análisis de logs del sistema (`/var/log/auth.log`).
- Monitorización de accesos remotos (SSH).
- Identificación de conexiones sospechosas.
- Revisión de procesos en ejecución.
- Análisis de puertos abiertos.

Durante el análisis realizado se identificó:

- Acceso no autorizado desde una IP externa.
- Uso de autenticación por contraseña.
- Posible manipulación de registros (logs).
- Indicios de persistencia en el sistema.

---

### 2.3 Contención

El objetivo de esta fase es limitar el impacto del incidente y evitar su propagación.

Las acciones aplicadas fueron:

- Aislamiento del sistema afectado de la red.
- Bloqueo de direcciones IP sospechosas.
- Desactivación de cuentas comprometidas.
- Implementación de reglas de firewall restrictivas.
- Restricción temporal de accesos remotos.

Se diferencian dos tipos de contención:

- **Contención a corto plazo:** aislamiento inmediato del sistema.
- **Contención a largo plazo:** aplicación de medidas permanentes de seguridad.

---

### 2.4 Erradicación

En esta fase se eliminan completamente las causas del incidente.

Medidas aplicadas:

- Eliminación de usuarios maliciosos.
- Revisión y eliminación de posibles rootkits.
- Limpieza del sistema de archivos sospechosos.
- Actualización del sistema operativo y servicios.
- Corrección de configuraciones inseguras.

Se prestó especial atención a:

- Servicios expuestos innecesariamente.
- Configuraciones débiles en SSH.
- Falta de control de accesos.

---

### 2.5 Recuperación

El objetivo es restaurar el sistema a un estado seguro y operativo.

Acciones realizadas:

- Restauración del sistema desde backups seguros.
- Cambio de credenciales de todos los usuarios.
- Reconfiguración de servicios críticos.
- Reapertura controlada del acceso remoto.
- Monitorización intensiva tras la recuperación.

Se verifica que el sistema:

- No presenta actividad sospechosa.
- Funciona correctamente.
- Está protegido frente a ataques similares.

---

### 2.6 Lecciones aprendidas

Tras el análisis del incidente, se identificaron los siguientes problemas:

- Uso de contraseñas débiles.
- Falta de autenticación segura (uso de claves SSH).
- Ausencia de monitorización continua.
- Configuración insegura de servicios expuestos.

Como mejora:

- Implementación de autenticación por clave pública.
- Restricción de acceso SSH.
- Monitorización activa del sistema.
- Aplicación de políticas de seguridad.

---

## 3. Respuesta ante un ataque similar

En caso de repetirse un incidente similar, la organización seguirá el siguiente procedimiento:

1. Detección del incidente mediante monitorización.
2. Bloqueo inmediato del origen del ataque.
3. Aislamiento del sistema afectado.
4. Análisis forense del incidente.
5. Eliminación de la amenaza.
6. Restauración desde copias de seguridad.
7. Refuerzo de medidas de seguridad.

Este enfoque reduce el tiempo de respuesta y minimiza el impacto en la organización.

---

## 4. Medidas de protección de datos

### 4.1 Copias de seguridad

- Realización de backups periódicos automatizados.
- Almacenamiento en ubicaciones seguras (externas o en la nube).
- Verificación de integridad de los datos.
- Pruebas de restauración.

---

### 4.2 Cifrado de datos

- Uso de protocolos seguros (SSH, HTTPS).
- Cifrado de datos sensibles almacenados.
- Protección de credenciales.
- Uso de algoritmos de cifrado robustos.

---

### 4.3 Control de accesos

- Aplicación del principio de mínimo privilegio.
- Uso de contraseñas robustas.
- Implementación de autenticación mediante claves SSH.
- Restricción de acceso por IP.
- Auditoría de accesos.

---

## 5. Implementación del SGSI (ISO 27001)

El Sistema de Gestión de Seguridad de la Información (SGSI) permite gestionar la seguridad de forma estructurada.

---

### 5.1 Análisis de riesgos

Se identificaron los siguientes riesgos:

- Accesos no autorizados.
- Exposición de servicios.
- Pérdida de datos.
- Instalación de malware.
- Escalada de privilegios.

Cada riesgo se evalúa en función de:

- Probabilidad
- Impacto

---

### 5.2 Políticas de seguridad

Se definen las siguientes políticas:

- Política de control de accesos.
- Política de contraseñas.
- Política de copias de seguridad.
- Política de gestión de incidentes.
- Política de uso aceptable de sistemas.

---

### 5.3 Tratamiento de riesgos

Medidas implementadas:

- Configuración de firewall.
- Desactivación de servicios innecesarios.
- Monitorización continua.
- Actualizaciones periódicas.
- Hardening del sistema.

---

### 5.4 Mejora continua

El SGSI se basa en un ciclo de mejora continua:

- Auditorías periódicas.
- Revisión de logs.
- Evaluación de vulnerabilidades.
- Actualización de controles de seguridad.
- Formación continua del personal.

---

## 6. Conclusión

La implementación de un plan de respuesta a incidentes basado en NIST, junto con un SGSI conforme a ISO 27001, permite a la organización mejorar significativamente su nivel de seguridad.

Estas medidas no solo permiten responder eficazmente ante incidentes, sino también prevenir futuros ataques, proteger la información crítica y garantizar la continuidad de los servicios.

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

* `fase1_1_logs_ssh.png`
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

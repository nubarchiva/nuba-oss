# Instalación de nubarchiva en Ubuntu 24.04 LTS

Este documento detalla los pasos necesarios para instalar y configurar nubarchiva en un servidor
Ubuntu 24.04 LTS (Noble Numbat).

> **Nota importante sobre Ubuntu 24.04**: A diferencia de versiones anteriores, Ubuntu 24.04 no
> incluye Tomcat 9 como paquete del sistema (solo ofrece Tomcat 10, que es incompatible con
> nubarchiva). Esta guía utiliza una instalación manual de Tomcat 9.

## Requisitos Previos

- Ubuntu 24.04 LTS (servidor o escritorio)
- 4 CPU / 8 GB RAM mínimo recomendado
- 20 GB de disco disponible
- Acceso root o sudo

Actualice el sistema antes de comenzar:

```bash
sudo apt update
sudo apt upgrade -y
```

## Configuración regional del sistema

Confirme que el locale `es_ES` está instalado.

```bash
locale -a | grep es_ES
```

La respuesta obtenida debe ser:

```text
es_ES.utf8
```

Si la respuesta está vacía, instale el locale `es_ES`:

```bash
sudo locale-gen es_ES.UTF-8
sudo update-locale
```

## Instalación de PostgreSQL

```bash
sudo apt install -y postgresql
```

Verificación:

```bash
sudo su - postgres -c "psql -c 'SELECT version();'"
```

Debería mostrar algo como:

```text
PostgreSQL 16.x ...
```

## Instalación de Java

nubarchiva necesita dos versiones de Java:

- **Java 8**: para Apache Solr 3.5
- **Java 11**: para Apache Tomcat 9

```bash
sudo apt install -y openjdk-8-jdk-headless openjdk-11-jdk-headless
```

Verificación:

```bash
java -version
```

```text
openjdk version "11.0.x" ...
```

```bash
/usr/lib/jvm/java-8-openjdk-amd64/bin/java -version
```

```text
openjdk version "1.8.0_xxx" ...
```

## Instalación de Apache Solr 3.5

Apache Solr 3.5 no está disponible en los repositorios oficiales de Ubuntu 24.04, por lo
que es necesario descargarlo manualmente.

```bash
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-apache-solr-3.5.0.tar.gz
tar xzf nuba-apache-solr-3.5.0.tar.gz
sudo mv apache-solr apache-solr-3.5.0 apache-solr-master /opt
```

Crear el usuario `solr`:

```bash
sudo useradd -r -s /bin/bash -d /opt/apache-solr solr
sudo chown -R solr:solr /opt/apache-solr-3.5.0 /opt/apache-solr-master
```

Cree el servicio de Solr:

```bash
sudo vi /etc/systemd/system/solr.service
```

```ini
[Unit]
Description=Apache Solr
After=network.target

[Service]
User=solr
PIDFile=/run/solr.pid
WorkingDirectory=/opt/apache-solr/server
ExecStart=/opt/apache-solr/server/start.sh
ExecStop=/bin/kill -TERM $MAINPID
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Inicie y habilite Solr:

```bash
sudo systemctl daemon-reload
sudo systemctl start solr
sudo systemctl enable solr
```

Verificación:

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8983/solr/
```

```text
200
```

## Instalación de Apache Tomcat 9

> **¿Por qué no `apt install tomcat9`?** Ubuntu 24.04 solo ofrece Tomcat 10 en sus repositorios.
> Tomcat 10 utiliza el namespace `jakarta.servlet`, mientras que nubarchiva utiliza `javax.servlet`.
> Son incompatibles. Es necesario instalar Tomcat 9 manualmente.

Descargue e instale Tomcat 9:

```bash
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.102/bin/apache-tomcat-9.0.102.tar.gz
sudo tar xzf apache-tomcat-9.0.102.tar.gz -C /opt
sudo mv /opt/apache-tomcat-9.0.102 /opt/tomcat9
```

Cree un usuario del sistema para Tomcat:

```bash
sudo useradd -r -s /bin/false -d /opt/tomcat9 tomcat
sudo chown -R tomcat:tomcat /opt/tomcat9
```

Verificación:

```bash
ls /opt/tomcat9/bin/startup.sh
```

```text
/opt/tomcat9/bin/startup.sh
```

## Configuración de la Base de Datos

Descargue y ejecute los scripts SQL que crean la base de datos `nubarchiva`, el esquema
`nuba00001`, las tablas y los datos iniciales de la aplicación.

```bash
mkdir nuba-sql && cd nuba-sql
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-sql/nuba.00.create_database.sql
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-sql/nuba.00.create_schema.sql
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-sql/nuba.01.create_table.sql
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-sql/nuba.02.create_fk.sql
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-sql/nuba.07.insert.general.sql
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-sql/nuba.08.insert.custom.sql
cat *.sql | sudo su - postgres -c psql
cd ..
```

> **⚠️ SEGURIDAD**: Los scripts SQL crean un usuario de base de datos y un usuario administrador
> de la aplicación con **contraseñas por defecto**.
> Es **imprescindible** cambiar ambas contraseñas
> antes de poner el sistema en producción.
> Consulte la sección
> [Seguridad post-instalación](#seguridad-post-instalación)
> al final de este documento.

Ejecute el siguiente comando para crear la tabla `institution` y registrar su archivo:

```bash
sudo su - postgres -c "psql -d nubarchiva -c \"
CREATE TABLE nuba00001.institution (
    idinstitution SERIAL PRIMARY KEY,
    code VARCHAR(10) NOT NULL,
    name VARCHAR(50) NOT NULL
);
INSERT INTO nuba00001.institution (code, name) VALUES ('nuba00001', 'Mi Archivo');
GRANT ALL ON nuba00001.institution TO nubauser;
GRANT USAGE, SELECT ON SEQUENCE nuba00001.institution_idinstitution_seq TO nubauser;
\""
```

> **Nota**: Sustituya `Mi Archivo` por el nombre de su institución o archivo.

Verificación:

```bash
sudo su - postgres -c "psql -d nubarchiva -c \"SELECT count(*) FROM nuba00001.users;\""
```

```text
 count
-------
     2
(1 row)
```

## Configuración de Tomcat

### Librerías

Copie las librerías necesarias al directorio `lib` de Tomcat:

```bash
mkdir nuba-lib && cd nuba-lib
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-app/lib/postgresql-42.3.10.jar
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-app/lib/commonj-1.1.1.jar
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-app/lib/foo-commonj-1.1.2.jar
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-app/lib/javax.activation-api-1.2.0.jar
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-app/lib/javax.annotation-api-1.2.jar
sudo cp *.jar /opt/tomcat9/lib/
cd ..
```

Verificación:

```bash
ls /opt/tomcat9/lib/postgresql*.jar
```

```text
/opt/tomcat9/lib/postgresql-42.3.10.jar
```

### Datasource (context.xml)

Configure el datasource editando el archivo `context.xml` de Tomcat:

```bash
sudo vi /opt/tomcat9/conf/context.xml
```

Agregue los dos `Resource` dentro del elemento `<Context>`:

```xml
<Context>

    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>WEB-INF/tomcat-web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <Resource name="jdbc/NUBADS"
              auth="Container"
              type="javax.sql.DataSource"
              maxTotal="100"
              minIdle="2"
              maxIdle="4"
              maxWaitMillis="10000"
              driverClassName="org.postgresql.Driver"
              username="nubauser"
              password="CAMBIAR_CONTRASEÑA"
              url="jdbc:postgresql://localhost:5432/nubarchiva?currentSchema=nuba00001"/>

    <Resource name="wm/default"
              auth="Container"
              type="commonj.work.WorkManager"
              factory="de.myfoo.commonj.work.FooWorkManagerFactory"
              minThreads="96"
              maxThreads="96"
              queueLength="256"/>
</Context>
```

> **⚠️ SEGURIDAD**: El valor de `password` en el `Resource` `jdbc/NUBADS` debe coincidir con la
> contraseña que haya asignado al usuario `nubauser` en PostgreSQL.

### Opciones de la JVM (setenv.sh)

A diferencia de Ubuntu 22.04 (donde se edita `/etc/default/tomcat9`), en la instalación manual se
crea el archivo `setenv.sh` dentro de Tomcat:

```bash
sudo vi /opt/tomcat9/bin/setenv.sh
```

```bash
JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"

JAVA_OPTS="-Djava.awt.headless=true"

# nubarchiva
JAVA_OPTS="${JAVA_OPTS} -Dhazelcast.phone.home.enabled=false"
JAVA_OPTS="${JAVA_OPTS} --illegal-access=warn"
JAVA_OPTS="${JAVA_OPTS} --add-modules java.se"
JAVA_OPTS="${JAVA_OPTS} --add-exports java.base/jdk.internal.ref=ALL-UNNAMED"
JAVA_OPTS="${JAVA_OPTS} --add-opens=java.base/java.nio=ALL-UNNAMED"
JAVA_OPTS="${JAVA_OPTS} --add-opens=java.base/sun.nio.ch=ALL-UNNAMED"
JAVA_OPTS="${JAVA_OPTS} --add-opens=java.management/sun.management=ALL-UNNAMED"
JAVA_OPTS="${JAVA_OPTS} --add-opens=jdk.management/com.sun.management.internal=ALL-UNNAMED"
```

Dele permisos de ejecución:

```bash
sudo chmod +x /opt/tomcat9/bin/setenv.sh
```

### Servicio systemd para Tomcat

Cree un servicio para que Tomcat se inicie automáticamente con el sistema:

```bash
sudo vi /etc/systemd/system/tomcat9.service
```

```ini
[Unit]
Description=Apache Tomcat 9
After=network.target postgresql.service

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="CATALINA_HOME=/opt/tomcat9"
Environment="CATALINA_BASE=/opt/tomcat9"

ExecStart=/opt/tomcat9/bin/startup.sh
ExecStop=/opt/tomcat9/bin/shutdown.sh

ReadWritePaths=/var/log/nubarchiva/

[Install]
WantedBy=multi-user.target
```

## Directorio de datos de nubarchiva

Cree la carpeta donde nubarchiva almacena los documentos adjuntos:

```bash
sudo mkdir -p /var/lib/nubarchiva/attachments
sudo chown -R tomcat:tomcat /var/lib/nubarchiva
```

## Logs de nubarchiva

Cree la carpeta de logs y configure sus permisos:

```bash
sudo mkdir -p /var/log/nubarchiva
sudo chown tomcat:tomcat /var/log/nubarchiva
```

## Instalación del archivo WAR de nubarchiva

Descargue y despliegue el archivo WAR:

```bash
wget https://releases.nubarchiva.org/nuba-2.24.7/nuba-app/nuba-web-2.24.7.war
sudo cp nuba-web-2.24.7.war /opt/tomcat9/webapps/nuba.war
sudo chown tomcat:tomcat /opt/tomcat9/webapps/nuba.war
```

## Arranque

Inicie y habilite los servicios:

```bash
sudo systemctl daemon-reload
sudo systemctl start tomcat9
sudo systemctl enable tomcat9
```

Espere unos 30 segundos para que nubarchiva se despliegue (el WAR es de ~108 MB).

Verificación:

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/nuba/
```

```text
200
```

Si se accede a [http://your-server:8080/nuba](http://your-server:8080/nuba) se muestra la página
de inicio de nubarchiva.

## Seguridad post-instalación

> **⚠️ IMPORTANTE**: Realice estos cambios **antes** de exponer el servidor a la red.

### 1. Cambiar la contraseña del usuario de base de datos

```bash
sudo su - postgres -c "psql -c \"ALTER USER nubauser WITH PASSWORD 'SU_NUEVA_CONTRASEÑA';\""
```

Actualice la contraseña también en `/opt/tomcat9/conf/context.xml` (atributo `password` del
Resource `jdbc/NUBADS`) y reinicie Tomcat:

```bash
sudo systemctl restart tomcat9
```

### 2. Cambiar la contraseña del administrador

Acceda a nubarchiva con las credenciales por defecto, vaya a
**Administración → Usuarios** y cambie la contraseña del usuario administrador.

### 3. Eliminar las aplicaciones de ejemplo de Tomcat

Tomcat incluye aplicaciones de administración y ejemplos que no deben estar accesibles en
producción:

```bash
sudo rm -rf /opt/tomcat9/webapps/ROOT
sudo rm -rf /opt/tomcat9/webapps/docs
sudo rm -rf /opt/tomcat9/webapps/examples
sudo rm -rf /opt/tomcat9/webapps/host-manager
sudo rm -rf /opt/tomcat9/webapps/manager
```

## Correspondencia de rutas: instalación manual vs. apt

Si encuentra documentación o guías que asumen una instalación de Tomcat 9 vía `apt-get`,
utilice esta tabla para traducir las rutas:

| Concepto    | Con apt (22.04)             | Manual (24.04)                  |
|-------------|-----------------------------|---------------------------------|
| Dir. base   | `/var/lib/tomcat9/`         | `/opt/tomcat9/`                 |
| Config.     | `/etc/tomcat9/`             | `/opt/tomcat9/conf/`            |
| context.xml | `/etc/tomcat9/context.xml`  | `/opt/tomcat9/conf/context.xml` |
| JAVA_OPTS   | `/etc/default/tomcat9`      | `/opt/tomcat9/bin/setenv.sh`    |
| Librerías   | `/var/lib/tomcat9/lib/`     | `/opt/tomcat9/lib/`             |
| Webapps     | `/var/lib/tomcat9/webapps/` | `/opt/tomcat9/webapps/`         |
| Servicio    | `systemctl restart tomcat9` | `systemctl restart tomcat9`     |

## Solución de problemas

### Tomcat no arranca

Revise los logs de Tomcat:

```bash
cat /opt/tomcat9/logs/catalina.out
```

**Error `java.lang.UnsupportedClassVersionError`**: Está ejecutando Tomcat con una versión de
Java incorrecta. Verifique que `setenv.sh` apunta a Java 11.

**Error `javax.naming.NameNotFoundException: Name [jdbc/NUBADS] is not bound`**: El `context.xml`
no contiene la configuración del datasource.
Revise el contenido de `/opt/tomcat9/conf/context.xml`.

### nubarchiva no carga (error de conexión a base de datos)

**Error `org.postgresql.util.PSQLException: Connection refused`**: PostgreSQL no está corriendo.

```bash
sudo systemctl start postgresql
```

**Error `password authentication failed for user "nubauser"`**: La contraseña en `context.xml` no
coincide con la del usuario en PostgreSQL.

### Solr no arranca

Revise los logs:

```bash
cat /opt/apache-solr/server/logs/*.log
```

**Error `java.lang.UnsupportedClassVersionError`**: Solr 3.5 requiere Java 8. Verifique que el
script `start.sh` usa la ruta correcta de Java 8 (`/usr/lib/jvm/java-8-openjdk-amd64`).

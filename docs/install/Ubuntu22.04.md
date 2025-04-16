# Instalación de nubarchiva en Ubuntu 22.04

Este documento detalla los pasos necesarios para instalar y configurar nubarchiva en un servidor
Ubuntu 22.04. Incluye la instalación de Tomcat, PostgreSQL y Apache Solr.

## Requisitos Previos

Antes de comenzar, asegúrese de que su sistema esté actualizado:

```bash
sudo apt update
sudo apt upgrade
```

## Configuración regional del sistema

Confirme que el locale `es_ES` está instalado.

```shell
locale -a | grep es_ES
```

La respuesta obtenida debe ser:

```text
es_ES.utf8
```

Si hemos obtenido una respuesta vacía, debe instalar el `locale` `es_ES`

```shell
sudo locale-gen es_ES.UTF-8
sudo update-locale
```

## Instalación de PostgreSQL

```bash
sudo apt install postgresql
```

Verificación

```bash
sudo su - postgres
psql
\q
exit
```

## Instalación de Apache Tomcat

```bash
sudo apt install openjdk-11-jdk-headless tomcat9
```

Verificación

```bash
java -version
```

```text
openjdk version "11.0.22" 2024-01-16
OpenJDK Runtime Environment (build 11.0.22+7-post-Ubuntu-0ubuntu222.04.1)
OpenJDK 64-Bit Server VM (build 11.0.22+7-post-Ubuntu-0ubuntu222.04.1, mixed mode, sharing)
```

Si se accede a [http://your-server:8080/](http://your-server:8080/) se muestra la página
"It works!" de Tomcat

## Instalación de Apache Solr 3.5

Apache Solr 3.5 no está disponible en los repositorios oficiales de Ubuntu 22.04, por lo
que es necesario descargarlo manualmente.

```bash
wget [https://releases.nubarchiva.es/solr/3.5.0/nuba-apache-solr-3.5.0.tar.gz](http://releases.nubarchiva.org/nuba-2.24.7/nuba-apache-solr-3.5.0.tar.gz)
tar xzf nuba-apache-solr-3.5.0.tar.gz
sudo mv apache-solr apache-solr-3.5.0 apache-solr-master /opt
```

Crear el usuario "solr"

```bash
sudo useradd -r -s /bin/bash -d /opt/apache-solr solr
sudo chown -R solr:solr /opt/apache-solr-3.5.0 /opt/apache-solr-master
```

Para su ejecución Apache Solr 3.5 necesita Java 8

```shell
sudo apt-get install openjdk-8-jdk-headless
```

Cree un script de inicio para Solr:

```bash
sudo vi /etc/systemd/system/solr.service
```

```unit file (systemd)
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

## Configuración de la Base de Datos

Ejecutar los scripts sql que crean la base de datos `nubarchiva`, el esquema `nuba00001`, las
tablas y contenidos usados por la aplicación.

```bash
cd nuba-sql
cat *.sql  | sudo su - postgres -c psql
```

## Configuración de Tomcat

Para que Tomcat pueda conectarse a PostgreSQL, es necesario instalar el driver JDBC
correspondiente.

```sh
sudo apt-get install tomcat9
```

Enlazar el archivo del driver JDBC al directorio de librerías de Tomcat:

```sh
sudo cp tomcat-lib/* /var/lib/tomcat9/lib
```

Configure el datasource en Tomcat:

```bash
sudo vi /etc/tomcat9/context.xml
```

Agregue la configuración de los siguientes `Resource` dentro del elemento `<Context>`:

```xml

<Context>
    <Resource name="jdbc/NUBADS"
              auth="Container"
              type="javax.sql.DataSource"
              maxTotal="100"
              minIdle="2"
              maxIdle="4"
              maxWaitMillis="10000"
              driverClassName="org.postgresql.Driver"
              username="nubauser"
              password="nuba_password"
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

En la configuración de las opciones inicio Tomcat, fichero `/etc/default/tomcat9`, modificar
JAVA_OPTS`

```shell
JAVA_OPTS="-Djava.awt.headless=true"

# nubarchiva configuration
JAVA_OPTS="${JAVA_OPTS} -Dhazelcast.phone.home.enabled=false"
JAVA_OPTS="${JAVA_OPTS} --illegal-access=warn"
JAVA_OPTS="${JAVA_OPTS} --add-modules java.se"
JAVA_OPTS="${JAVA_OPTS} --add-exports java.base/jdk.internal.ref=ALL-UNNAMED"
JAVA_OPTS="${JAVA_OPTS} --add-opens=java.base/java.nio=ALL-UNNAMED"
JAVA_OPTS="${JAVA_OPTS} --add-opens=java.base/sun.nio.ch=ALL-UNNAMED"
JAVA_OPTS="${JAVA_OPTS} --add-opens=java.management/sun.management=ALL-UNNAMED"
JAVA_OPTS="${JAVA_OPTS} --add-opens=jdk.management/com.sun.management.internal=ALL-UNNAMED"
```

```shell
sudo mkdir /etc/systemd/system/tomcat9.service.d
echo -e "[Service]\nReadWritePaths=/var/log/nubarchiva/" | sudo tee /etc/systemd/system/tomcat9.service.d/logging-allow.conf
sudo systemctl daemon-reload
```

Reinicie Tomcat para aplicar los cambios:

```sh
sudo systemctl restart tomcat9
```

## Logs de nubarchiva

Cree la carpeta de logs y configure sus permisos:

```bash
sudo mkdir /var/log/nubarchiva
sudo chown tomcat:tomcat /var/log/nubarchiva
```

## Instalación del archivo WAR de nubarchiva

Copie el archivo WAR de nubarchiva al directorio de despliegue de Tomcat:

```sh
sudo cp nuba-web-2.24.7.war /var/lib/tomcat9/webapps/nuba.war
sudo chown tomcat:tomcat  /var/lib/tomcat9/webapps/nuba.war
```

Si se accede a [http://your-server:8080/nuba](http://your-server:8080/nuba) se muestra la página
de inicio de nubarchiva

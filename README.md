# OpenCMS

OpenCMS es un marco de administración de contenido de código abierto basado en Java .  Ayuda a los administradores de contenido a crear y mantener sitios web de manera rápida y eficiente . Desarrollado por Alkacon Software, admite varios idiomas y múltiples sitios a escala empresarial.

OpenCMS es un CMS basado en Java, por lo que debe tener Java instalado para poder usarlo. Para saber que java debemos instalar en nuestro sistema.

## Instalar Java
```sh
$ java -version
```
De las opciones que nos devuelve, vamos a instalar la primera opción que nos aparece.
```sh
$ sudo apt install default-jre
```
#### Comprobar si tenemos Java
Comprobamos introduciendo de nuevo el comando anterior, pero esta vez observamos que lo que devuelve es diferente
```sh
openjdk version "11.0.4" 2019-07-16
OpenJDK Runtime Environment (build 11.0.4+11-post-Ubuntu-1ubuntu218.04.3)
OpenJDK 64-Bit Server VM (build 11.0.4+11-post-Ubuntu-1ubuntu218.04.3, mixed mode, sharing)
```
#### Instalar Java Compiler
Repetimos el proceso con el comando javac que es el compilador de java, que necesitaremos para que los entornos virtuales corran en java
```sh
$ javac -version
```
Instalamos el default-jdk
```sh
$ sudo apt install default-jdk
```
#### Comprobamos
Cuando comprobamos, nos imprime la versión la versión que hemos instalado
```sh
javac 11.0.4
```
## Instalar TOMCAT
OpenCMS también requiere del servidor web Tomcat, por lo que vamos a instalarlo desde su página oficial.
https://tomcat.apache.org/download-80.cgi
Elegimos el archivo Core tar.gz y para descargar
```sh
$ wget http://apache.uvigo.es/tomcat/tomcat-8/v8.5.49/bin/apache-tomcat-8.5.49.tar.gz
```
Descomprimimos con tar
```sh
$ tar xvzf apache-tomcat-9.0.29.tar.gz
```
Creamos un directorio en /opt y movemos el archivo que acabamos de descomprimir a dicho directorio.
```sh
$ sudo mv apache-tomcat-9.0.29 /opt/tomcat9
```
Crearemos un usuario tomcat
```sh
$ sudo useradd -r tomcat9 --shell /bin/false
```
Vamos a darle a Tomcat el control del directorio
```sh
$ sudo chown -R tomcat9 /opt/tomcat9
```
Configuramos tomcat. Abrimos el archivo de configuración tomcat-users.xml
```sh
$ sudo vim /opt/tomcat9 /conf/tomcat-users.xml
```
Añadimos las siguientes líneas. Deben de colocarse antes de la línea </tomcat-users>
```sh
<role rolename = "manager-gui" />
<role rolename = "admin-gui" />
<usuario nombre de usuario = "admin" contraseña = "usuario" roles = "manager-gui, admin-gui" />
```
Creamos una cuenta de servidor para Tomcat
```sh
$ sudo vim /etc/systemd/system/tomcat.service
```
Añadimos las siguientes líneas
```sh
[Unit]
Description=Tomcat9
After=network.target
[Service]
Type=forking
User=tomcat9
Group=tomcat9
Environment=CATALINA_PID=/opt/tomcat9/tomcat9.pid
Environment=JAVA_HOME=/usr/lib/jvm/java-8-oracle/
Environment=CATALINA_HOME=/opt/tomcat9
Environment=CATALINA_BASE=/opt/tomcat9
Environment="CATALINA_OPTS=-Xms512m -Xmx512m"
Environment="JAVA_OPTS=-Dfile.encoding=UTF-8 -Dnet.sf.ehcache.skipUpdateCheck=true -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+UseParNewGC"
ExecStart=/opt/tomcat9/bin/startup.sh
ExecStop=/opt/tomcat9/bin/shutdown.sh
[Install]
WantedBy=multi-user.target
```
Reiniciamos el servicio 
```sh
$ sudo systemctl restart tomcat.service
$ sudo systemctl start tomcat.service
$ sudo systemctl enable tomcat.service
```
Arrancamos el servicio
```sh
$ cd /opt/tomcat9/bin
$ sudo ./startup.sh
```
Comprobar escribiendo en el navegador
http://localhost:8080

## Instalar Gestor de BD Mysql MariaDB
OpenCMS también requiere un servidor de base de datos, y el servidor de base de datos MariaDB es un excelente lugar para comenzar. Para instalarlo, ejecute los siguientes comandos.
```sh
$ sudo apt-get install mariadb-server mariadb-client
```
Después de instalar MariaDB, los siguientes comandos se pueden usar para detener, iniciar y habilitar el servicio MariaDB para que siempre se inicie cuando se inicia el servidor. 
```sh
$ sudo systemctl stop mariadb.service
$ sudo systemctl start mariadb.service
$ sudo systemctl enable mariadb.service
```

Debemos despues proteger el servidor MariaDB creando una contraseña de root y rechazando el acceso root remoto. 
```sh
$ sudo mysql_secure_installation
```
Cuando se le solicite, responda las siguientes preguntas

    - Ingrese la contraseña actual para root (ingrese para ninguno): solo presione Enter
    - Establecer contraseña de root? [S / n]: S
    - Nueva contraseña: Ingrese la contraseña / usuario
    - Reingrese la nueva contraseña: Repita la contraseña / usuario
    - ¿Eliminar usuarios anónimos? [S / n]: S 
    - ¿No permitir el inicio de sesión root de forma remota? [S / n]: S
    - ¿Eliminar la base de datos de prueba y acceder a ella? [S / n]: S
    - ¿Recargar tablas de privilegios ahora? [S / n]: S

Reiniciamos el servicio MariaDB
```sh
$  sudo systemctl restart mysql.service
```
Iniciar sesión en el servidor de la base de datos. Cuando se nos solicite una contraseña, escribimos la contraseña raíz que creamos anteriormente.
```sh
$  sudo mysql -u root -p
```
Creamos una base de datos de MariaDB a la que llamaremos opencms
```sh
MariaDB [(none)]> CREATE DATABASE opencms;
```
Creamos un usuario en la base de datos 
```sh
MariaDB [(none)]> CREATE USER ' opencmsuser '@'localhost' IDENTIFIED BY ' usuario ';
```
Otorgamos al usuario acceso completo a la base de datos.
```sh
MariaDB [(none)]> GRANT ALL ON opencms .* TO ' opencmsuser '@'localhost' IDENTIFIED BY ' usuario' WITH GRANT OPTION;
```
Guardamos los cambios y salimos de la base de datos.
```sh
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> EXIT;
```
## Instalar OpenCMS
Para descargar e instalar OpenCMS, descargamos enlace de su página oficial
http://www.opencms.org/en/modules/downloads/begindownload.html?id=44d15ab0-58f6-11e8-a3d4-7fde8b0295e1
Descargamos con wget
```sh
$ wget http://www.opencms.org/downloads/opencms/opencms-10.5.4.zip
```
Descomprimimos el archivo que acabamos de descargar
```sh
$ unzip opencms-10.5.4.zip
```
Mover el archivo .war que se nos ha generado de la descarga y moverlo al directorio de tomcat llamado webapps
```sh
$ sudo mv opencms.war /opt/tomcat9/webapps/
```
Reiniciamos el servidor Tomcat.
```sh
$ sudo systemctl restart tomcat.service
```
Ahora nos vamos al navergador para comenzar la instalación de OpenCMS
http://localhost:8080/opencms/setup

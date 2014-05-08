Mostraremos la instalación en un sistema operativo Linux Debian 7 a través de paquetes. A continuación se detallan los pasos que se van a seguir:

1. Instalar FreeRADIUS.
2. Configurar FreeRADIUS.
3. Descargar daloRADIUS.
4. Instalar dependencias daloRADIUS.
5. Instalar y configurar daloRADIUS.
6. Test de funcionamiento.


--------------------------------------------------------------------------------------------------------------------
1.El primer paso es instalar FreeRADIUS desde paquetes. Para ello lanzamos el siguiente comando como root desde una terminal y confirmamos la instalación

	apt-get update
	apt-get install freeradius freeradius-mysql freeradius-utils
	
--------------------------------------------------------------------------------------------------------------------
2. Ahora vamos a dejar preparado nuestro FreeRADIUS para que conecte posteriormente con la base de datos MySQL. Los datos de conexión a la base de datos dejaremos los predeterminados que aparecen en /etc/freeradius/sql.conf y activaremos el soporte sql para autenticación y cuentas. Ejecutamos los siguientes comandos como root:
	
	cd /etc/freeradius
	vim sites-enabled/default
	
Para activar el soporte SQL editaremos el fichero radius.conf y descomentaremos la línea $INCLUDE sql.conf. Para ello como root ejecutamos el siguiente comando
	
	vim radius.conf
	Descomentamos la línea $INCLUDE sql.conf
	
--------------------------------------------------------------------------------------------------------------------
3. El siguiente paso es descargar daloRADIUS desde mi repositorio. Para esta operación ejecutamos los siguientes comandos como root:
	
		cd /usr/local/src
		wget https://codeload.github.com/dm94/TutoRadius/zip/master
	
--------------------------------------------------------------------------------------------------------------------
4. Vamos a instalar el servicio Web, el de base de datos y un paquete para gestión de imágenes. Como root ejecutamos el siguiente comando:
	
		apt-get install apache2 php5 php5-gd php-pear php-db libapache2-mod-php5 php-mail php5-mysql mysql-server
	
--------------------------------------------------------------------------------------------------------------------
5. Una vez instaladas las dependencias vamos a descomprimir la aplicación en el directorio de nuestro servicio Web y crearemos la base de datos con las tablas pertinentes. Como root ejecutamos los siguientes comandos
	
	cd /var/www
	tar xzf /usr/local/src/<FUENTES_daloRADIUS.tar.gz>
	mv -f <DIRECTORIO_daloRADIUS> daloradius/
	chown -R www-data:www-data daloradius
	chmod 644 daloradius/library/daloradius.conf.php
	vim daloradius/library/daloradius.conf.php
	
		CONFIG_DB_USER --> radius
		CONFIG_DB_PASS --> radpass
		
Ahora vamos a crear la base de datos con su esquema completo. Para ello ejecutamos los siguientes comandos como root:

	mysqladmin -u root -p create radius
	cd /var/www/daloradius/contrib/db
	mysql -u root -p radius < fr2-mysql-daloradius-and-freeradius.sql
	mysql -u root -p
		GRANT ALL ON radius.* TO radius@localhost IDENTIFIED BY 'radpass';
		quit
		
--------------------------------------------------------------------------------------------------------------------
6. Ahora vamos a probar la instalación que hemos realizado, pero antes de nada vamos a reiniciar los servicios Web y FreeRADIUS para que se apliquen todos los cambios hechos anteriormente. Para ello ejecutamos los siguientes comandos como root
	
	service freeradius restart
	service apache2 restart
	
A través de un explorador Web vamos a acceder a la gestión de nuestro daloRADIUS para crear un usuario y realizar una prueba desde consola. Para ello ingresamos en la dirección http://localhost/daloradius. Las credenciales son administrator/radius
	
	Accedemos al apartado management
	Damos ha "New User - Quick add"
	Le ponemos un nombre y una contraaseña y damos a "apply"
	
Ahora que tenemos el usuario vamos a realizar un test de conexión desde la consola. Para ello ejecutamos el siguiente comando como root y si nos devuelve Access-Accept es que todo ha ido bien
	
	radtest "usuario" "contraseña" 127.0.0.1 1581 testing123
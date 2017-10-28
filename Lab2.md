Trabajos realizados

Máquina con ip pública

Issue#10

# Para instalar el repositorio ius ejecutamos el comando: "yum -y install https://centos7.iuscommunity.org/ius-release.rpm"
# Instalamos php 7.1 con "yum -y install php71u php71u-fpm"
# Modificamos el archivo /etc/php.ini y agregamos la línea: "cgi.fix_pathinfo=0". Esto por motivos de seguridad, para que usuarios no puedan ejecutar scripts que no tienen permitido ejecutar
# Modificamos el archivo /etc/php-fpm.d/www.conf:
### listen lo cambiamos de TCP socket a unix socket ("listen 127.0.0.1:9000" -> "listen /var/run/php-fpm/php-fpm.sock"). Esto es más seguro puesto que el unix socket es un archivo que está en la misma máquina, y cuyos permisos pueden restringirse, a diferencia del tcp socket que se escucha desde localhost, y otros sin autorización también lo pueden leer
### Modificamos "user" y "group", y cambiamos sus valores por "nginx". Esto es para que el proceso que crea, lee y modifica el socket sea de nginx. Del mismo modo, "listen.owner" y "listen.group" también se les da el valor de "nginx", de manera que el socket se cree con nginx como su dueño, y por lo tanto, tengan permiso de lectura y escritura
### Respecto a las variables que comienzan con pm, consisten en los valores con que el administrador de procesos (pm -> process manager) utiliza para su funcionamiento. max_children es el número máximo de "hijos" que puede crear el administrador. start_servers es el número de hijos que se crean una vez que el servicio inicia. min_spare_servers y max_spare_servers son, respectivamente, el mínimo y el máximo número de hijos en espera (idle). Si hay menos hijos en espera que min_spare_servers, se crean nuevos hijos, y si hay más que max_spare_servers, entonces se mata algunos de los hijos. max_requests es el número máximo de "requests" que puede hacer un hijo antes de ser eliminado, y es útil para evitar memory leaks. Los valores de todas estas variables dependen del número de usuarios que visitarán la página, y cuantas solicitudes php se hacen por segundo, por lo que dar un número ahora sería difícil
### Respecto a "slowlog", es el documento de logs de php para solicitudes lentas, o de larga duración. Como interesa saber las razones de que una solicitud sea lenta, y arreglarlo, es una herramienta fundamental en el proceso de debugging.
# Ejecutamos "systemctl start php-fpm" para iniciar php, y lo habilitamos con "systemctl enable php-fpm"

Issue#11

# Copiamos /etc/nginx/sites-availables/test.conf en /etc/nginx/sites-availables/opencart.conf y procedemos a modificar:
### Cambiamos el valor de root a /srv/www/opencart/html
### Agregamos index.php al parametro index
### Descomentamos el bloque de php (el de nginx) y modificamos el valor de fast_cgi por unix:/var/run/php-fpm/php-fpm.sock. También cambiamos el valor de "fastcgi_param SCRIPT_FILENAME" por "$document_root$fastcgi_script_name"
# Creamos enlace simbólico a opencart.conf y lo guardamos en /etc/nginx/sites-enabled/
# Borramos el enlace a test.conf en la misma carpeta
# Movemos /srv/www/test a /srv/www/opencart
# Reiniciamos nginx

Issue#12

# Instalamos el repositorio de Percona con "yum -y install http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm"
# Instalamos Percona con "yum -y install Percona-Server-server-57"
# Ejecutamos "systemctl start mysql" para iniciar el motor de base de datos
# Buscamos la clave de root@localhost para mysql con el comando: "grep 'temporary password' /var/log/mysqld.log"
# Nos conectamos a mysql con "mysql -u root -p" y la contraseña que acabamos de conseguir
# Cambiamos la contraseña con "ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewSecurePassword*';"
# Creamos base de datos con "CREATE DATABASE opencart;"
# Creamos usuario a la vez que le damos acceso a la base de datos con: "GRANT ALL ON opencart.* TO 'ocuser' IDENTIFIED BY 'MyNewSecurePassword*';"
# Ejecutamos "FLUSH PRIVILEGES;"
# Salimos con "exit"

Issue#13

# Instalamos unzip y wget con yum
# Descargamos un zip con los archivos de opencart desde https://github.com/opencart/opencart/archive/master.zip usando wget, y lo guardamos en /srv/www/opencart/html/
# Abrimos el zip con el comando " unzip master.zip 'opencart-master/upload/* ". Luego copiamos el archivo opencart-master/upload/config-dist.php a opencart-master/upload/config.php y el archivo opencart-master/upload/admin/config-dist.php por opencart-master/upload/admin/config.php
# Cambiamos el user y group de todos los archivos en opencart-master/upload a nginx:nginx usando chown
# Movemos los contenidos de opencart-master/upload/ a /srv/www/opencart/html/
# Borramos master.zip y los demás archivos de opencart-master
# Arreglamos los contextos de selinux con "/sbin/restorecon -vR /srv/www/opencart/html/* "
# Ejecutamos "setsebool -P httpd_unified 1" y "setsebool -P httpd_can_network_connect_db 1" para cambiar unos booleano de selinux que permitirán a php acceder a los datos
# Instalamos la extensiones php71u-json, php71u-gd y php71u-mysqlnd
# Vamos a la página grupo09.mosorio.me y nos encontramos con el instalador de opencart. Le damos CONTINUE
# Revisamos que todos los tickets estén en verde y le damos a CONTINUE
# Rellenamos los datos de acceso a la base de datos, como el username (ocuser) y el nombre de la base de datos (opencart). Creamos un usuario admin y le damos una contraseña y un correo. Luego le damos a CONTINUE.
# Borramos la carpeta /srv/www/opencart/html/install y todos los archivos y subcarpetas dentro

Issue#14

# Instalamos ansible via "yum -y install ansible"
# Modificamos /etc/ansible/hosts y agregamos las ips de ambas máquinas
# Creamos las llaves ssh de maquina1 y maquina2, y las agregamos al authorized_keys de cada una
# Creamos la carpeta /etc/ansible/group_vars y el archivo /etc/ansible/group_vars/servers
# Le decimos a ansible que siempre intente conectarse como usuario root, al modificar el archivo /etc/ansible/group_vars/servers
# Ejecutamos "mkdir /root/ansible" para crear la carpeta donde guardaremos los playbooks
# Creamos los playbooks y los ejecutamos con ansible-playbooks nombre-playbook

Issue#15

# Volvemos a hacer las tareas del lab1 en la nueva máquina, exceptuando lo relacionado a nginx

Issue#17

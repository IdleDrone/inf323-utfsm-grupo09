Trabajo realizado:

En máquina con ip pública

-Ingreso desde mi computador con la clave entregada a traves de la ip pública
-Cambio la contraseña usando passwd como root
-Cambio el hostname con "hostnamectl set-hostname grupo09-m1" (Se cambiará al salir y volver a conectarme)

Issue#3
-Configuro los repositorios para que apunten a ftp.inf.utfsm.cl, pero ya estaba hecho (incluso epel)
-Actualizo la máquina con "yum -y update"
-Instalo vim, bash-completion y bash-completion-extras, pero ya estaban instalados (wow)
-Instalo setroubleshoot-server para arreglar problemas de selinux en el futuro

Issue#4
-Modifico /etc/ssh/sshd_config y agrego la línea "PermitRootLogin without-password"
-Modifico el archivo .ssh/authorized_keys y agregar mi llave. NO borro llaves de mosorio ni ydossow
-Reinicio sshd.service para cargar la nueva configuración


Issue#2
-Configuro interfaz eth0 y eth1, pero ya estaban bien configuradas (que eficiencia)

Issue#9
-Reviso selinux con "getenforce". Enforcing, así que está bien

Issue#7
-Instalo firewalld con "yum -y install firewalld", pero ya estaba instalado
-Ejecuto "systemctl status firewalld". Se confirma que está corriendo
-Ejecuto "firewall-cmd --get-zones" para ver las zonas existentes. La zona "public" ya existe, así que agrego la zona "private" con "firewall-cmd --permanent --new-zone=private"
-Agrego el servicio HTTP con "firewall-cmd --permanent --zone=public --add-service=http". Lo mismo para HTTPS, cambiando http por https
-Recargo las reglas con "firewall-cmd --reload" para activar los cambios hechos

Issue#5
-Ejecuto "lsblk" para encontrar el disco adicional. Se llama /dev/sdb
-Ejecuto "parted /dev/sdb mklabel gpt" para crear la lista de particiones del disco
-Creo una única partición que ocupe todo el disco con "parted -a opt /dev/sdb mkpart primary ext4 0% 100%"
-Creo el filesystem de la nueva partición con "mkfs.ext4 -L webserver /dev/sdb1"
-Monto el filesystem en /srv/ con el comando "mount /dev/sdb1 /srv"
-Modifico el archivo /etc/fstab/ para que el nuevo disco se monte automáticamente al rebootear: Agrego la línea "LABEL=webserver /srv ext4 defaults 0 0"

Issue#6
-Creo el archivo /etc/yum.repos.d/nginx.repo, luego copio y pego los datos que me indican en la página oficial
-Ejecuto "yum -y install nginx" para instalar nginx
-Inicio nginx con "systemctl start nginx", y lo habilito (para que inicie junto con los otros servicios al encender la máquina) con "systemctl enable nginx"

Issue#8

-Ejecuto "mkdir -p /srv/www/test/html" para crear la ruta donde guardar nuestra página"
-Cambio los permisos con "chmod -R 755 /srv/www" (para que otros puedan entrar en la carpeta y ver la página)
-Creo el archivo /srv/www/test/html/index.html y lo relleno con un "Hello world"
-Creo las carpetas /etc/nginx/sites-available y /etc/nginx/sites-enabled
-Edito el archivo /etc/nginx/nginx.conf: agrego la ruta "/etc/nginx/sites-enabled/\*.conf" para que encuentre el Serverblock de la página. También agrego la línea "server_names_hash_bucket_size 64;" para aumentar el tamaño del buffer que lee la url
-Copio la plantilla de serverblock desde /etc/nginx/conf.d/default.conf y lo pego en /etc/nginx/sites-available/test.conf
-Modifico /etc/nginx/sites-available/test.conf :  cambio el server_name por "grupo09.mosorio.me" y la ruta en que busca la página por "/srv/www/test/html"
-Creo un enlace simbólico a /etc/nginx/sites-available/test.conf y lo guardo en /etc/nginx/sites-enabled
-Restauro los contextos de selinux del archivo /var/ww/test/html/index.html con /sbin/restorecon (para esto usé el comando "sealert -a /var/log/audit/audit.log" y ejecuté el comando que decía)
-Reinicio nginx con "systemctl restart nginx"

Issue#1

-Instalo ntp via yum, pero ya está instalado
-Reviso el estado de ntp con "systemctl status ntpd" y funciona correctamente
-Abro el archivo "/etc/ntp.conf" para ver que "pools" está usando. Apunta a las "pools" del departamento, así que está ok

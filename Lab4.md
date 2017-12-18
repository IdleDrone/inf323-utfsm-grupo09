Issue#21

Instalar docker
yum install docker
systemctl start docker
systemctl status docker
systemctl enable docker

Actualizar kernel
wget https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
gpg --quiet --with-fingerprint RPM-GPG-KEY-elrepo.org
fingerprint correcta
rpm --import RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum install yum-plugin-fastestmirror
instalado
yum install kernel-ml --enablerepo=elrepo-kernel
grep '^menuentry' /boot/grub2/grub.cfg
vemos la posicion del kernel que deseamos, en este caso es 0
vim /etc/default/grub y ponemos GRUB_DEFAULT=0
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot

Cambiar storage driver
usaremos overlay2 porque lo recomendó el profesor
systemctl stop docker
Modificamos /etc/docker/daemon.json y agregamos:  "storage-driver": "overlay2" y "storage-opts": ["overlay2.override_kernel_check=true"]
systemctl start docker


Issue#22

Descargar jenkins y crear el container:
docker pull jenkins
docker run -d -u root --name jenkins -p 8080:8080 -v /root/jenkins:/var/jenkins_home:z -t jenkins
firewall-cmd --permanent --zone=public --add-port=8080/tcp
firewall-cmd --reload
vim /etc/nginx/sites-availables/jenkins.conf y agregar:

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
upstream app {
  server 127.0.0.1:8080;
}
server {
  listen 80;
  server_name jenkins.grupo09.mosorio.me;

  location / {
    proxy_pass http://app;
  }
}
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

setsebool -P httpd\_can\_network_connect 1
setsebool -P httpd\_can\_network_relay 1

Ingresamos a jenkins.grupo09.mosorio.me, luego buscamos la password en el archivo que nos indica, reemplazando /var/jenkins\_home por /root/jenkins (o sea, en /root/jenkins/secrets/initialAdminPassword) e introducimos la contraseña
Install recommended plugins
Creamos primer admin user:

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Username: caraya
Password: MySecretPassword*
Confirm password: MySecretPassword*
Full name: Cristian Araya
E-mail address: caraya@alumnos.inf.utfsm.cl
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
y luego START USING JENKINS (yay)
y luego updateamos a la nueva version de jenkins desde la misma página y restart

Instalamos los plugins para jenkins
Vamos a jenkins.grupo09.mosorio.me y seleccionamos Manage Jenkins -> Manage plugins -> Available y buscamos Docker plugin y Git plugin (Git plugin ya estaba instalado, así que instalamos Docker plugin)
Install without restart
Go back to the top page

Issue#25
Creamos certificado ssl con certbot certonly para jenkins
Modificamos /etc/nginx/sites-available/jenkins.conf:
Básicamente le metemos SSL y arreglamos el proxy provisorio. Queda así
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
upstream jenkins {
  server 127.0.0.1:8080 fail_timeout=0;
}

server {
  listen 80;
  server_name jenkins.grupo09.mosorio.me;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  server_name jenkins.grupo09.mosorio.me;

Blablabla SSL lo mismo de opencart.conf pero con 2 cambios:
ssl_certificate /etc/letsencrypt/live/jenkins.grupo09.mosorio.me/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/jenkins.grupo09.mosorio.me/privkey.pem;

location / {
    proxy_set_header        Host $host:$server_port;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header        X-Forwarded-Proto $scheme;

    proxy_pass              http://jenkins;
    proxy_read_timeout      90;

    proxy_redirect          http:// https://;

    # Required for new HTTP-based CLI
    proxy_http_version 1.1;
    proxy_request_buffering off;
    proxy_buffering off; # Required for HTTP-based CLI to work over SSL
    # workaround for https://issues.jenkins-ci.org/browse/JENKINS-45651
    add_header 'X-SSH-Endpoint' 'jenkins.grupo09.mosorio.me:50022' always;
  }
}
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
systemctl restart nginx

Issue#23
yum update because why not
yum install git
agregar llaves a las deploy keys del repo

en el repo:
crear app de django
crear Dockerfile y ponerle la info necesaria (obviamente), o sea, copiamos el template de https://www.caktusgroup.com/blog/2017/03/14/production-ready-dockerfile-your-python-django-app/ y le cambiamos los my_project por mysite
creamos el archivo requirements.txt y le agregamos los módulos que requiere, como django por ejemplo
creamos el archivo docker-entrypoint.sh y le ponemos el blabla
le ponemos el entrypoint al dockerfile
creamos el docker-compose y declaramos el postgres
lo metemos en el Dockerfile
Modificamos el settings.py

en grupo09-m1:
git clone git@github.com:IdleDrone/inf323-utfsm-grupo09.git
cd inf323-utfsm-grupo09/mysite
docker build -t mysyte .
docker login --username=idledrone
docker tag 77f7a3b4848d idledrone/mysite:first
docker push idledrone/mysite:first

firewall-cmd --permanent --zone=internal --add-interface=docker0
firewall-cmd --permanent --add-port=2376/tcp
firewall-cmd --reload

Issue#23 Denuevo desde 0

curl -L https://github.com/docker/compose/releases/download/1.18.0-rc2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
creamos una carpeta de nombre grupo09app en el repo. Entramos con cd
Siguiendo este tutorial: https://docs.docker.com/compose/django/ creamos el Dockerfile, requirements.txt y docker-compose.yml y los guardamos en la carpeta. en docker-compose.yml le agregamos ":Z" a la declaración del volúmen para no tener problemas con SELinux
Ejecutamos "/usr/local/bin/docker-compose run web django-admin.py startproject grupo09app ."
Editamos grupo09app/settings.py y reemplazamos la sección DATABASES
Ejecutamos "/usr/local/bin/docker-compose up"
Funciona bien, ahora vams a hacer denuevo el build pero ahora con docker build:
docker login --username=idledrone
"docker build -t grupo09app ."
Buscamos el ID de grupo09app con "docker images"
"docker tag e48e23d73856 idledrone/grupo09app:latest"
"docker push idledrone/grupo09app"
Luego en el server:
docker pull idledrone/grupo09app:latest

Issue#23 AAAAAAAAAAAAAAA

Seguiré esta guia https://www.capside.com/labs/deploying-full-django-stack-with-docker-compose/ y espero que esta vez funcione
Creamos las carpetas y el docker-compose, además del env
Creamos docker-entrypoint-initdb.d/grupo09_web.sh y en él creamos el usuario y la base de datos
Le damos permiso de ejecución

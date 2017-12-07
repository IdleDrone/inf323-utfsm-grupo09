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
Modificamos /etc/docker/daemon.json y agregamos:  "storage-driver": "overlay" y "storage-opts": ["overlay2.override_kernel_check=true"]
systemctl start docker


Issue#22

Descargar jenkins y crear el container
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

/* Esto creo que solo lo hice en el tutorial, y que es parte del issue 23 o 24
Cofigurar los plugins
Manage Jenkins -> Configure System -> Add a new Cloud -> Docker
Aparece un nuevo apartado para docker, y lo rellenamos de información

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Name: docker-agent
Docker URL: vacío
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

Configurar imágen
Images dropdown -> Docker Image
Docker Image -> benhall/dind-jenkins-agent
label -> docker-agent
Credentials -> username: jenkins, password: jenkins
Select jenkins/*******
Container Settings -> Volumes: /var/run/docker.sock:/var/run/docker.sock
Save*/

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
Modificmos el settings.py

en grupo09-m1:
git clone git@github.com:IdleDrone/inf323-utfsm-grupo09.git

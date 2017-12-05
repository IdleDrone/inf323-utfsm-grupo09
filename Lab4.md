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

Descargar jenkins
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

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

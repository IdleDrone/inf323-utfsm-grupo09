Issue#21

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

docker info | grep 'Storage Driver'
dice que está usando 'devicemapper'
uname -a
dice que tenemos kernel 3, por lo que usamos overlay en lugar de overlay2
Modificamos /etc/docker/daemon.json y agregamos:  "storage-driver": "overlay"
systemctl restart docker

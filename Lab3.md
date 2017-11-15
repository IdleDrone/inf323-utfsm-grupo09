# Trabajos realizados

# Issue#20

### Instalamos percona en la máquina nueva de la misma forma que en la máquina principal
### Modificamos el archivo "/etc/percona-server.conf.d/mysqld.cnf" de ambas máquinas
##### En la máquina 1 agregamos la línea "server-id=1" y en la máquina 2 agregamos la línea "server-id=2"
##### Agregamos la línea "log-bin=/var/log/mysql/mysql-bin.log" en ambas máquinas
### Instalamos "percona-xtrabackup-24" con yum en ambas máquinas
### Creamos la carpeta ""/root/percona-backup" donde guardaremos el respaldo de la base de datos (en master)
### Ejecutamos "innobackupex --user=root --password=MyNewSecurePassword* /root/percona-backup/"
### Ejecutamos "innobackupex --user=root --password=MyNewSecurePassword* --aply-log /root/percona-backup/$TIMESTAMP", donde $TIMESTAMP es el nombre de la carpeta dentro de /root/percona-backup que se creó con el comando anterior

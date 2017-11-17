# Trabajos realizados

# Issue#20

### Instalamos percona en la máquina nueva de la misma forma que en la máquina principal
### Modificamos el archivo "/etc/percona-server.conf.d/mysqld.cnf" de ambas máquinas
##### En la máquina 1 agregamos la línea "server-id=1" y en la máquina 2 agregamos la línea "server-id=2"
##### Agregamos la línea "log-bin=bin.log" en ambas máquinas
##### Agregamos la línea "log-bin-index=bin.index" en ambas maquinas
### Reiniciamos mysqld en ambas máquinas
### Instalamos "percona-xtrabackup-24" con yum en ambas máquinas
### Creamos la carpeta "/root/percona-backup" donde guardaremos el respaldo de la base de datos (en master)
### Ejecutamos "innobackupex --user=root --password=MyNewSecurePassword* /root/percona-backup/"
### Ejecutamos "innobackupex --user=root --password=MyNewSecurePassword* --apply-log /root/percona-backup/$TIMESTAMP", donde $TIMESTAMP es el nombre de la carpeta dentro de /root/percona-backup que se creó con el comando anterior
### Editamos en ambas máquinas el archivo "/etc/percona-server.conf.d/mysqld.cnf" y agregamos las líneas:
##### [client]
##### user=root
##### password=MyNewSecurePassword*
### Ejecutamos "rsync -avpP -e ssh /root/percona-backup/$TIMESTAMP 10.10.15.226:/var/lib/percona-backup" para copiar la data del Master al Slave
### Ahora en el Slave: detenemos mysql con "systemctl stop mysqld"
### Respaldamos la configuración actual con "mv /var/lib/mysql /var/lib/mysql_bak"
### Ejecutamos "mv /var/lib/percona-backup/$TIMESTAMP /var/lib/mysql" para cargar la nueva configuración
### Cambiamos el user y el group de mysql con "chown -R mysql:mysql /var/lib/mysql"
### Restauramos los contextos de selinux con "/sbin/restorecon -Rv /var/lib/mysql"
### Eliminamos el directorio /var/lib/percona-backup
### En el Master: en mysql: Le damos permisos para replicación al Slave con "GRANT REPLICATION SLAVE ON *.*  TO 'rep'@'10.10.15.226' IDENTIFIED BY 'MyNewSlavePassword*';"
### En el Slave: nos conectamos al Master con "mysql --host=10.10.15.237 --user=rep --password=MyNewSlavePassword*" y revisamos que efectivamente tenemos permisos con "SHOW GRANTS;" dentro de mysql
### En el Slave:
##### Reiniciamos mysqld
##### Ingresamos a mysql
##### Ejecutamos "CHANGE MASTER TO MASTER_HOST='10.10.15.237', MASTER_USER='rep', MASTER_PASSWORD='MyNewSlavePassword*', MASTER_LOG_FILE='bin-000001', MASTER_LOG_POS=0;"
##### Iniciamos el Slave con "START SLAVE;"

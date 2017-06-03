# Práctica 5
## Replicación de Bases de Datos MySQL

### Crear un tar con ficheros locales y copiarlos en un equipo remoto

Ya vimos en la práctica 2 cómo crear un tar.gz con un directorio de un equipo y dejarlo
en otro mediante ssh.

Para ello, vimos que deberemos indicar al comando tar que queremos que use stdout
como destino y mandar con una pipe la salida al ssh. Éste debe coger la salida del tar
y escribirla en un fichero. El comando quedaría:

**tar czf - directorio | ssh equipodestino 'cat > ~/tar.tgz'**

De esta forma en el equipo de destino tendremos creado el archivo tar.tgz
También vimos cómo ejecutar de una forma similar el comando scp. Sin embargo,
esto que puede ser útil en un momento dado, no nos servirá para sincronizar grandes
cantidades de información. En el caso de BD, hay mejores formas de hacerlo.



### Crear una BD e insertar datos

Para el resto de la práctica debemos crearnos una BD en MySQL e insertar algunos
datos. Así tendremos datos con los cuales hacer las copias de seguridad. En todo
momento usaremos la interfaz de línea de comandos del MySQL:

**mysql -uroot -p**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-1.jpg)


Enter password: <TECLEAR LA CLAVE SI NO ES VACÍA>
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.1.44 Source distribution
Type 'help;' or '\h' for help. Type '\c' to clear the current
input statement.


**mysql> create database contactos;**

Query OK, 1 row affected (0,00 sec)

**mysql> use contactos;**

Database changed

**mysql> show tables;**

Empty set (0,00 sec)

**mysql> create table datos(nombre varchar(100),tlf int);**

Query OK, 0 rows affected (0,01 sec)

**mysql> show tables;**

+---------------------+


| Tables_in_contactos |

+---------------------+

| datos |

+---------------------+

1 row in set (0,00 sec)
mysql> insert into datos(nombre,tlf) values ("pepe",95834987);
Query OK, 1 row affected (0,00 sec)

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-2.jpg)


**mysql> select * from datos;**


+---------+-----------+

| nombre | tlf |

+---------+-----------+

| pepe | 95834987 |

+---------+-----------+

3 rows in set (0,00 sec)

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-3.jpg)

**mysql> quit**

Bye

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-4.jpg)





### Replicar una BD MySQL con mysqldump

MySQL ofrece la una herramienta para clonar las BD que tenemos en nuestra
maquina. Esta herramienta es mysqldump.

La sintaxis de uso es:

**mysqldump ejemplodb -u root -p > /root/ejemplodb.sql**

Esto puede ser suficiente, pero tenemos que tener en cuenta que los datos pueden
estar actualizándose constantemente en el servidor de BD principal. En este caso,
antes de hacer la copia de seguridad en el archivo .SQL debemos evitar que se
acceda a la BD para cambiar nada.
Así, en el servidor de BD principal (maquina1) hacemos:

**mysql -u root –p**

**mysql> FLUSH TABLES WITH READ LOCK;**

**mysql> quit**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-5.jpg)

Ahora ya sí podemos hacer el mysqldump para guardar los datos. En el servidor
principal (maquina1) hacemos:

**mysqldump contactos -u root -p > /tmp/contactos.sql**

Como habíamos bloqueado las tablas, debemos desbloquearlas (quitar el “LOCK”):

**mysql -u root –p**

**mysql> UNLOCK TABLES;**

**mysql> quit**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-6.jpg)


Ya podemos ir a la máquina esclavo (maquina2, secundaria) para copiar el archivo
.SQL con todos los datos salvados desde la máquina principal (maquina1):

**scp maquina1:/tmp/ejemplodb.sql /tmp/**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-7.jpg)


y habremos copiado desde la máquina principal (1) a la máquina secundaria (2) los
datos que hay almacenados en la BD.

Con el archivo de copia de seguridad en el esclavo ya podemos importar la BD
completa en el MySQL. Para ello, en un primer paso creamos la BD:

**mysql -u root –p**

**mysql> CREATE DATABASE ‘ejemplodb’;**

**mysql> quit**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-8.jpg)


Y en un segundo paso restauramos los datos contenidos en la BD (se crearán las
tablas en el proceso):

**mysqldump contactos -u root -p < /tmp/contactos.sql**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-9.jpg)




### Replicación de BD mediante una configuración maestro-esclavo

La opción anterior funciona perfectamente, pero es algo que realiza un operador a
mano. Sin embargo, MySQL tiene la opción de configurar el demonio para hacer
replicación de las BD sobre un esclavo a partir de los datos que almacena el maestro.

Se trata de un proceso automático que resulta muy adecuado en un entorno de
producción real. Implica realizar algunas configuraciones, tanto en el servidor principal
como en el secundario.

A continuación se detalla el proceso a realizar en ambas máquinas, para lo cual,
supondremos que partimos teniendo clonadas las base de datos en ambas máquinas:

Lo primero que debemos hacer es la configuración de mysql del maestro. Para ello
editamos, como root, el **/etc/mysql/my.cnf** para realizar las modificaciones que se
describen a continuación.

Comentamos el parámetro bind-address que sirve para que escuche a un servidor:

**#bind-address 127.0.0.1**

Le indicamos el archivo donde almacenar el log de errores. De esta forma, si por
ejemplo al reiniciar el servicio cometemos algún error en el archivo de configuración,
en el archivo de log nos mostrará con detalle lo sucedido:

**log_error = /var/log/mysql/error.log**

Establecemos el identificador del servidor.

**server-id = 1**

El registro binario contiene toda la información que está disponible en el registro de
actualizaciones, en un formato más eficiente y de una manera que es segura para las
transacciones:

**log_bin = /var/log/mysql/bin.log**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-10.jpg)


Guardamos el documento y reiniciamos el servicio:

**/etc/init.d/mysql restart**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-11.jpg)


Si no nos ha dado ningún error la configuración del maestro, podemos pasar a hacer
la configuración del mysql del esclavo (vamos a editar como root el /etc/mysql/my.cnf).

La configuración es similar a la del maestro, con la diferencia de que el server-id en
esta ocasión será 2.

Reiniciamos el servicio en el esclavo:
**/etc/init.d/mysql restart**

y una vez más, si no da ningún error, habremos tenido éxito.
Podemos volver al maestro para crear un usuario y darle permisos de acceso para la replicación.
Entramos en mysql y ejecutamos las siguientes sentencias:

**mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-12.jpg)


**mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%'**

**IDENTIFIED BY 'esclavo';**

**mysql> FLUSH PRIVILEGES;**

**mysql> FLUSH TABLES;**

**mysql> FLUSH TABLES WITH READ LOCK;**

Para finalizar con la configuración en el maestro, obtenemos los datos de la base de
datos que vamos a replicar para posteriormente usarlos en la configuración del
esclavo:

**mysql> SHOW MASTER STATUS;**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-13.jpg)


Volvemos a la máquina esclava, entramos en mysql y le damos los datos del maestro.

En el entorno de mysql ejecutamos la siguiente sentencia (ojo con la IP, "master_log_file" y
del "master_log_pos" del maestro):
**mysql> CHANGE MASTER TO MASTER_HOST='192.168.31.200',
MASTER_USER='esclavo', MASTER_PASSWORD='esclavo',
MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=501,
MASTER_PORT=3306;**

Por último, arrancamos el esclavo y ya está todo listo para que los demonios de
MySQL de las dos máquinas repliquen automáticamente los datos que se
introduzcan/modifiquen/borren en el servidor maestro:

**mysql> START SLAVE;**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-14.jpg)


Ahora, podemos hacer pruebas en el maestro y deberían replicarse en el esclavo
automáticamente.

Por último, volvemos al maestro y volvemos a activar las tablas para que puedan
meterse nuevos datos en el maestro:

**mysql> UNLOCK TABLES;**

Ahora, si queremos asegurarnos de que todo funciona perfectamente y que el esclavo
no tiene ningún problema para replicar la información, nos vamos al esclavo y con la
siguiente orden:

**mysql> SHOW SLAVE STATUS\G**

Revisamos si el valor de la variable “Seconds_Behind_Master” es distinto de “null”. En
ese caso, todo estará funcionando perfectamente.

Si no es así, es que hay algún error y los valores de las demás variables indicarán cuál
es el problema y cómo arreglarlo para que todo funcione perfectamente. A
continuación vemos que el valor de la variable “Seconds_Behind_Master” es 0, por lo
que no hay ningún error y todo funciona como debe:

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p5-15.jpg)

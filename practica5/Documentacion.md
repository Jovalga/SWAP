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



![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p4-1.jpg)







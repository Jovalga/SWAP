# Practica 6
## Discos en RAID

### Configuración del RAID por sofware

Partimos de una máquina virtual ya instalada y configurada a la
que, estando apagada, añadiremos dos discos del mismo tipo y capacidad.

Ahora arrancamos la máquina y entramos para instalar el software necesario para
configurar el RAID:

**sudo apt-get install mdadm**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-1.jpg)

Debemos buscar la información (identificación asignada por Linux) de ambos discos:

**sudo fdisk -l**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-2.jpg)

Ahora ya podemos crear el RAID 1, usando el dispositivo /dev/md0, indicando el
número de dispositivos a utilizar (2), así como su ubicación:

**sudo mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-3.jpg)

En este punto, el dispositivo se habrá creado con el nombre /dev/md0, sin embargo,
en cuanto reiniciemos la máquina, Linux lo renombrará y pasará a llamarlo
/dev/md127.

Una vez creado el dispositivo RAID, y como aún no habremos reiniciado la máquina,
usaremos /dev/md0 para darle formato:

**sudo mkfs /dev/md0**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-4.jpg)

Por defecto, mkfs inicializa un dispositivo de almacenamiento con formato ext2.
Ahora ya podemos crear el directorio en el que se montará la unidad del RAID:

**sudo mkdir /dat**

**sudo mount /dev/md0 /dat**

Podemos comprobar que el proceso se ha realizado adecuadamente, y también los
parámetros con los que Linux ha conseguido montarlo usando la orden:

**sudo mount**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-5.jpg)

Para comprobar el estado del RAID, ejecutaremos:
**sudo mdadm --detail /dev/md0**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-6.jpg)

Para finalizar el proceso, conviene configurar el sistema para que monte el dispositivo
RAID creado al arrancar el sistema. Para ello debemos editar el archivo /etc/fstab y
añadir la línea correspondiente para montar automáticamente dicho dispositivo.
Conviene utilizar el identificador único de cada dispositivo de almacenamiento en lugar
de simplemente el nombre del dispositivo (aunque ambas opciones son válidas). Para
obtener los UUID de todos los dispositivos de almacenamiento que tenemos, debemos
ejecutar la orden:

**ls -l /dev/disk/by-uuid/**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-7.jpg)



![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-8.jpg)



![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-9.jpg)
![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-10.jpg)
![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p6-11.jpg)





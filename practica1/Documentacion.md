# Práctica 1
## Preparación de las herramientas

Lo primero que debemos hacer es instalar apache y sql, podemos ejecutar el siguiente comando:
sudo apt-get install apache2 mysql-server mysql-client

Una vez instalado apache comprobaremos la versión del servidor ejecutando:
apache2 -v
Obtenemos los siguiente:

 ![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/practica1-1.jpg)


Y para ver si está en ejecución:
ps aux | grep apache
Obtenemos lo siguiente:


Finalmente creamos en /var/www/ el archivo hola.html, en su interior escribimos lo siguiente:

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/practica1-3.jpg)


Y vemos si mediante la orden curl podemos acceder desde un servidor al otro:

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/practica1-4.jpg)

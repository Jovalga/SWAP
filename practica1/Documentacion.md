# Práctica 1

Lo primero que debemos hacer es instalar apache y sql, podemos ejecutar el siguiente comando:
sudo apt-get install apache2 mysql-server mysql-client

Una vez instalado apache comprobaremos la versión del servidor ejecutando:
apache2 -v
Obtenemos los siguiente:


Y para ver si está en ejecución:
ps aux | grep apache
Obtenemos lo siguiente:


Finalmente creamos en /var/www/ el archivo hola.html, en su interior escribimos lo siguiente:


Y vemos si mediante la orden curl podemos acceder desde un servidor al otro:

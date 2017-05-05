# Práctica 3
## Balanceo de carga

Para comenzar he creado las dos nuevas máquinas virtuales. Para ello he duplicado
dos veces la primera, y asegurándome de que la máquina balanceadora no tiene apache
(o simplemente que no lo esté ejecutando). Por lo que ya tenemos las cuatro máquinas como se indica
en el enunciado del guión.


**NGINX**

Una vez creado el balanceador he ejecutado las órdenes para instalar
nginx:
**sudo apt-get update && sudo apt-get dist-upgrade && sudo apt-get
autoremove**
**sudo apt-get install nginx**
**sudo service nginx start**


Una vez hecho esto tenemos que modificar el fichero de
configuración **/etc/nginx/conf.d/default.conf.** dentro de la máquina
m3 (el balanceador) quedando así:



Ahora comprobamos que podemos poner en marcha el servicio nginx:


Y por último nos queda comprobar que funciona, por ello vamos a crear el archivo
/var/www/index.html en las máquinas 1 y 2, y en su interior vamos a escribir 
algo distintivo para que cuando le mandemos una petición desde la máquina exterior
(máquina 4) mediante el comando curl al balanceador (máquina 3), éste reparta las 
peticiones entre las dos máquinas (máquina 1 y máquina 2).
El resultado es el siguiente:



Vemos en la imagen anterior que accede tanto a la máquina 1 como a la máquina 2,
lo que nos indica que el balanceador funciona correctamente mediante nginx.
Con este ejemplo hemos usado balanceo mediante el algoritmo de “round-robin”, es decir,
con la misma prioridad para todos los servidores.



**HAPROXY**

Pasamos a haproxy. Lo instalamos con la orden:
**sudo apt-get install haproxy**

Pasamos a modificar y con ello a configurar el archivo **/etc/haproxy/haproxy.cfg**
quedando así:

Ahora lanzamos el servicio haproxy mediante el comando (parar antes nginx si se está
usando mediante *sudo service nginx stop*):
**sudo /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg**

Seguidamente probamos mediante **curl http://192.168.21.134 && curl http://192.168.21.134**
si el balanceo se ejecuta correctamente. El resultado es el siguiente:


Y vemos en la imagen anterior que el balanceo se produce satisfactoriamente.




Nos falta someter al balanceador a una alta carga, para ello vamos a usar
desde la máquina 4 Apache Benchmark (ab) que es una utilidad que se instala junto
con el servidor Apache y permite comprobar el rendimiento de cualquier servidor web.
Para utilizarlo debemos ejecutar la siguiente orden:
**ab -n 1000 -c 10 http://192.168.2.134/index.html**  (1000 veces)

El resultado de nginx es el siguiente:



El resultado de haproxy es el siguiente:














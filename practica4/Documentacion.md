# Práctica 4
## Asegurar la granja web

### Instalar un certificado SSL autofirmado para configurar el acceso por HTTPS

Comenzamos generando e instalando un certificado autofirmado.

Para generar un certificado SSL autofirmado en Ubuntu Server solo debemos activar
el módulo SSL de Apache, generar los certificados y especificarle la ruta a los
certificados en la configuración. Así pues, como root ejecutaremos:
**a2enmod ssl**

**service apache2 restart**

**mkdir /etc/apache2/ssl**

**openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout**

**/etc/apache2/ssl/apache.key- -out /etc/apache2/ssl/apache.crt**

Nos pedirá una serie de datos para configurar el dominio.

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p4-1.jpg)

Editamos el archivo de configuración del sitio default-ssl:
**nano /etc/apache2/sites-available/default-ssl**

Y agregamos estas lineas debajo de donde pone SSLEngine on:
**SSLCertificateFile /etc/apache2/ssl/apache.crt**
**SSLCertificateKeyFile /etc/apache2/ssl/apache.key**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p4-2.jpg)

Activamos el sitio default--ssl y reiniciamos apache:
**a2ensite default-ssl**
**service apache2 reload**

Una vez reiniciado Apache, accedemos al servidor web mediante el protocolo HTTPS
y veremos que en la barra de dirección sale en rojo el https, ya que se trata de un
certificado autofirmado.

Activamos el sitio default--ssl y reiniciamos apache:
**a2ensite default-ssl**
**service apache2 reload**




### Configuración del cortafuegos

A continuación se mostrará cómo utilizar la herramienta para establecer ciertas
reglas y filtrar algunos tipos de tráfico, o bien controlar el acceso a ciertas páginas:
Toda a información sobre la herramienta está disponible en su página de manual y
usando la opción de ayuda:
man iptables
iptables –h

Para comprobar el estado del cortafuegos, debemos ejecutar:
**iptables –L –n -v**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p4-3.jpg)


Para lanzar, reiniciar o parar el cortafuegos, y para salvar las reglas establecidas hasta
ese momento, ejecutaremos respectivamente:
*service iptables start
service iptables restart
service iptables stop
service iptables save*

También se puede parar el cortafuegos y eliminar al mismo tiempo todas sus reglas:
*iptables -F
iptables -X
iptables –t nat -F
iptables –t nat -X
iptables –t mangle -F
iptables –t mangle -X
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT*

Para denegar cualquier tráfico de información, podemos hacer:
*iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables –L –n -v*

Para bloquear el tráfico de entrada, podemos hacer:
*iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
iptables -A INPUT -m state --state NEW,ESTABLISHED -j ACCEPT
iptables –L –n -v*

Bloquear todo el tráfico ICMP (ping), para evitar ataques como el del ping de la
muerte:
*iptables -A INPUT -p icmp --icmp-type echo-request -j DROP*

Abrir el puerto 22 para permitir el acceso por SSH:
*iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p udp --sport 22 -j ACCEPT*

Abrir los puertos HTTP/HTTPS (80 y 443) para configurar un servidor web:
*iptables -A INPUT -m state --state NEW -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -m state --state NEW -p tcp --dport 443 -j ACCEPT*

Abrir el puerto 53 para permitir el acceso a DNS:
*iptables -A INPUT -m state --state NEW -p udp --dport 53 -j ACCEPT
iptables -A INPUT -m state --state NEW -p tcp --dport 53 -j ACCEPT*

Bloquear todo el tráfico de entrada desde una IP:
*iptables -I INPUT -s 150.214.13.13 -j DROP*

Bloquear todo el tráfico de salida hacia una IP:
*iptables -I OUTPUT -s 31.13.83.8 -j DROP*

Evitar el acceso a www.facebook.com especificando el nombre de dominio:
*iptables -A OUTPUT -p tcp -d www.facebook.com -j DROP*


Lo habitual es crear un script que se ejecute en el arranque del sistema. Veamos a
continuación un ejemplo de script para la configuración básica de una máquina Linux:

(1) Se eliminan todas las reglas que hubiera para hacer la configuración limpia:
**iptables -F**
**iptables -X**

(2) Establecer las políticas por defecto (denegar todo el tráfico):
**iptables -P INPUT DROP**
**iptables -P OUTPUT DROP**
**iptables -P FORWARD DROP**

(3) Permitir cualquier acceso desde localhost (interface lo):
**iptables -A INPUT -i lo -j ACCEPT**
**iptables -A OUTPUT -o lo -j ACCEPT**

(4) Abrir el puerto 22 para permitir el acceso por SSH:
**iptables -A INPUT -p tcp --dport 22 -j ACCEPT**
**iptables -A OUTPUT -p udp --sport 22 -j ACCEPT**

(5) Abrir los puertos HTTP/HTTPS (80 y 443) de servidor web:
**iptables -A INPUT -m state --state NEW -p tcp --dport 80 -j ACCEPT**
**iptables -A INPUT -m state --state NEW -p tcp --dport 443 -j ACCEPT**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p4-scriptbasico.jpg)


Por último, conviene comprobar el funcionamiento del cortafuegos recién configurado.
Para ello, pediremos al sistema que nos muestre qué puertos hay abiertos y qué
demonios o aplicaciones los tienen en uso. Para ello, utilizaremos la orden netstat
como se muestra a continuación:
**netstat -tulpn**

Por ejemplo, para asegurarnos del estado (abierto/cerrado) del puerto 80, podemos
ejecutar:
**netstat -tulpn | grep :80**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p4-netstat.jpg)




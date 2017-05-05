# Practica 2
## Clonar la información de un sitio web

Instalamos rsync mediante sudo apt-get install rsync
Para probar el funcionamiento del rsync vamos a clonar una carpeta cualquiera. Por
ejemplo, para clonar la carpeta con el contenido del servidor web principal, en la
máquina 2 (secundaria) ejecutaremos:
  **rsync -avz -e ssh ipmaquina1:/var/www/ /var/www/**

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p2-1.jpg)


Vemos en la imagen anterior que se ha conectado sin pedir la contraseña de la maquina 1,
lo normal es que nos pida la contraseña de la máquina 1 antes de la sincronización.
Esto es debido a que lo hemos congifurado para que entre estas máquinas haya "confianza".
Para conseguir esto hay que seguir estos pasos:

- Mediante ssh-keygen, podemos generar la clave, con la opción -t para el tipo de clave.
  Así, en la máquina secundaria ejecutaremos:
    **ssh-keygen -b 4096 -t rsa**
    
- A continuación deberemos copiar la clave pública al equipo remoto (máquina principal)
  añadiéndola al fichero ~/.ssh/authorized_keys, que deberá tener permisos 600 (por
  defecto esto estará bien configurado; sólo si nos da algún error debemos hacerlo):
     **chmod 600 ~/.ssh/authorized_keys**
  
- Para hacer la copia de la clave existe una forma sencilla y elegante. Se trata de usar el
  comando ssh-copy-id, que viene integrado con el comando ssh. Lo que haremos es
  copiarla a la máquina principal (a la que querremos acceder luego desde la
  secundaria). Ejecutamos en la máquina 2:
    **ssh-copy-id IPmaquina1**
    
    
        
Con respecto a crontab, para establecer una tarea en cron que se ejecute cada hora para mantener
actualizado el contenido del directorio /var/www entre las dos máquinas habría que escribir:
(Observar la última sentencia, en la que tenemos escrita la orden rsync)

![img](https://github.com/Jovalga/SWAP/blob/master/Imagenes/p2-1.jpg)

  

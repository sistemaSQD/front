![Sistema para la gestión y monitorización de squid!](https://www.sistemasqd.nat.cu/static/media/SQD.f7a0d058.png "SQD")
# Guía de instalación de SQD
## IMPORTANTE

1. Se recomienda instalar el sistema en ubuntu 18.04 o 20.04 ya que incluye mongodb en sus repos, puede usar debian pero necesita instalar Mongodb de forma manual.
2. Esta guía fue probada en Ubuntu 18.04 y 20.04, y Debian 10(en este caso instalando mongodb manualmente)
3. Para esta guía de instalación se asume lo siguiente:
   * El dominio en el cual se va a desplegar el sistema es **sqd.demo.cu**, usted debe de cambiarlo según su dominio.
   * El usuario que se usa para el despliegue de SQD no es root pero pertenece al grupo sudo por lo que puede escalar sus permisos, dicho usuario en este caso es uno llamado **administrator** especificado durante el proceso de instalación del sistema operativo usado, usted lo cambia según el usuario que usted use.  
     
## PROCESO DE INSTALACION  
### Nota:
>  Si su conexión no es mediante proxy salte los pasos hasta "Instalar dependencias"

### Conexión mediante proxy
Si su conexión es mediante proxy debe de asegurarse que los comando funcionen correctamente, para ello puede ejecutar los siguientes comandos teniendo presente si esa conexión mediante proxy es con o sin usuario.     
Para estos comandos se asume que el usuario es **miusuario**, el password es **P1ssw0rd**, la ip del servidor proxy a usar es **192.168.43.60** y que el puerto es **3128**, usted debe de sustituirlos por su datos.
#### proxy con usuario
    export http_proxy=http://miusuario:P1ssw0rd@192.168.43.60:3128
    export https_proxy=http://miusuario:P1ssw0rd@192.168.43.60:3128
#### proxy sin usuario
    export http_proxy=http://192.168.43.60:3128
    export https_proxy=http://192.168.43.60:3128

### Instalar dependencias

    sudo apt install git curl mongodb nginx squidclient ufw

Instalar node (Seleccionar la última versión estable, para cuando se hizo esta guía fue v16.13.1)

    curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
    source ~/.profile
    nvm ls-remote
    nvm install 16.13.1
    node -v
    npm -v

Clonar repos de git de SQD e iniciar el backend

    sudo mkdir /opt/SQD && sudo chown -R administrator /opt/SQD && cd /opt/SQD
    git clone https://github.com/sistemaSQD/api.git
    git clone https://github.com/sistemaSQD/front.git
    cd /opt/SQD/api
    npm i
    npm i -g pm2
    pm2 start /opt/SQD/api/SQD-api.js
    pm2 start /opt/SQD/api/handlers/SQD-bot.js
    pm2 save
    pm2 startup

Si al ejecutar el comando **pm2 startup** le pide ejecutar un comando como el mostrado a continuación, entonces copie ese comando y ejecutelo para poder levantar los procesos de SQD automáticamente al reiniciar el servidor. Ese comando varía en dependencia del usuario utilizado y de la versión de Node instalada.

    sudo env PATH=$PATH:/home/administrator/.nvm/versions/node/v16.13.1/bin /home/administrator/.nvm/versions/node/v16.13.1/lib/node_modules/pm2/bin/pm2 startup systemd -u administrator --hp /home/administrator

Modificar el archivo de llamada al backend según el dominio que usted está usando

    cd /opt/SQD/front
    nano env-config.js
        window._env_ = {
            REACT_APP_ENDPOINT: 'http://sqd.demo.cu:4000',
        }

Crear el VirtulHost para acceder a la interfaz de usuario

    sudo nano /etc/nginx/sites-available/demosqd
        server {
            listen 80;
            listen [::]:80;

            root /opt/SQD/front;
            index index.html;

            server_name sqd.demo.cu;

            location / {
                    try_files $uri $uri/ =404;
            }
        }
    sudo ln -s /etc/nginx/sites-available/demosqd /etc/nginx/sites-enabled/
    sudo nginx -t
        nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
        nginx: configuration file /etc/nginx/nginx.conf test is successful
    sudo systemctl restart nginx
    systemctl status nginx

Configurar conrtafuegos para permitir las conexiones al sistema

    sudo ufw app list
    sudo ufw allow 'Nginx HTTP'
    sudo ufw allow 4000
    sudo ufw status
    sudo ufw enable    #Ejecutar este comando si el estado es Status: inactive

En este momento usted puede acceder a:   
https://sqd.demo.cu    
> Recuerde que esta dirección cambia según la que usted ha usado

Una vez que acceda, el sistema le solicita la licencia correspondiente, para lo cual usted debe de registrarse en la web del proyecto https://www.sistemasqd.nat.cu y solicitarnos la licencia utilizando el formulario disponible en la web cuando acceda con sus datos de registro, debe de enviar el código que se muestra en la interfaz de usuario de SQD y el nombre de su empresa. Si la licencia que solicita es DEMO el sistema le genera automaticamente la licencia y usted la puede descargar y subirla al sistema SQD para comenzar la prueba, en caso de solicitar una licencia de pago deberá de esperar a nuestra aprobación.  

Una vez registrada la licencia que le enviaremos, los datos de acceso por defecto son:   
Usuario:  ***admin***    
Password: ***SQD***
## IMPORTANTE
Para poder gestionar y monitorear el servidor proxy que usted va a registrar primero debe de modificar el archivo de configuración de squid de dicho servidor proxy para permitir el acceso de SQD al administrador de cache de squid. Para el siguiente ejemplo se asume que el password que se le pone al administrador de cache de squid es **P1ssw0rd** y que la ip de la pc donde se instaló SQD es **192.168.43.30**

    nano /etc/squid/squid.conf
        cachemgr_passwd P1ssw0rd all
        acl manager proto cache_object
        acl sistemaSQD src 192.168.43.30/32
        http_access allow manager localhost
        http_access allow manager sistemaSQD
        http_access deny manager

Reiniciar el squid para aplicar el cambio

    service squid restart

Listo ya puede registrar su proxy para ser gestionado y monitoreado por SQD

### Recuerde que usted puede aportar también en este sistema para que se incluyan nuevas funcionalidades que sean de su interés

# Nota IMPORTANTE  

## Si al registrase en nuestra web el correo de verificación no le llega es muy probable que sea porque nuestra ip está dentro de un rango de ip de Cuba puestas en lista negra por http://www.sorbs.net/lookup.shtml y por http://www.spamrats.com. Lamentamos no poder resolver esta situación nosotros mismos ya que dependemos de nuestro proveedor para poderlo resolver. Usted puede agregar temporalmente nuestro dominio a su lista blanca en su servidor de correo para poder completar el registro. 
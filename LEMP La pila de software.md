# LEMP
La pila de software LEMP es un grupo de elementos de software que puede utilizarse para ofrecer páginas web y aplicaciones web dinámicas escritas en PHP. El acrónimo describe un sistema operativo Linux, con un servidor web Nginx (que se pronuncia ​como “Engine-X”). Los datos backend se almacenan en la base de datos de MySQL y el procesamiento dinámico se gestiona a través de PHP

## SSH
SSH es una herramienta esencial para ser un administrador de sistemas experto.
SSH, o Secure Shell, es un protocolo que se utiliza para iniciar sesión de forma segura en sistemas remotos. Es la forma más común de acceder a servidores Linux remotos.

### Sintaxis básica
Para conectarse a un sistema remoto mediante SSH, usaremos el comando ssh. El formato más básico del comando es:
```bash
$ ssh remote_host
```
Si su nombre de usuario es diferente en el sistema remoto, puede especificarlo usando esta sintaxis:
```bash
$ ssh remote_username@remote_host
```

### ¿Cómo funciona ssh?
SSH funciona mediante la conexión de un programa cliente a un servidor ssh, llamado sshd.

En la sección anterior, ssh era el programa cliente. El servidor ssh ya está en ejecución en el remote_host que se especificó.

En su servidor, el servidor sshd ya debe estar en ejecución. Si no es así, es posible que deba acceder a su servidor mediante una consola basada en la web o una consola serie local.

El proceso que debe iniciar un servidor ssh depende de la distribución de Linux que esté utilizando.

En Debian, para iniciar el servidor ssh, debe escribir lo siguiente:

```bash
$ sudo systemctl enable ssh --now
```

### Configurar ssh
```bash
# /etc/ssh/sshd_config
Port 22
LoginGraceTime 120
PermitRootLogin no
StrictModes yes
```

## Uncomplicated Firewall
Uncomplicated Firewall (ufw, y gufw - una versión con interfaz gráfica) — Uncomplicated Firewall (ufw) es un frontal para iptables y está particularmente bien adaptado para los cortafuegos basados en host. Ufw proporciona un marco para la gestión de netfilter así como una interfaz de línea de comandos para manipular el cortafuegos.

### Instalación
```bash
$ sudo nala/apt install ufw
```

### Configuración
Aviso: Si estás configurando via SSH, querrás abrir el paso a SSH antes de habilitar el cortafuegos. Si se interrumpe tu conexión antes de abrir SSH podrías quedarte aislado de tu sistema.

En primer lugar se debe habilitar el cortafuegos escribiendo:
```bash
$ sudo ufw enable
```

En segundo lugar, se deben configurar los ajustes por defecto. Para la mayoría de usuarios los siguientes ajustes por defecto pueden ser suficientes.
```bash
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
```

A continuación, es recomendable verificar que el cortafuegos está activado escribiendo:
```bash
$ sudo ufw status verbose
```

### Rango de puertos
Se pueden especificar también rangos de puertos, un ejemplo para tcp sería:
```bash
$ sudo ufw allow 1000:2000/tcp
```
y para udp:
```bash
$ sudo ufw allow 1000:2000/udp
```
### Dirección IP
```bash
$ sudo ufw allow from 111.222.333.444
```

### Eliminar Reglas
Las reglas pueden ser eliminadas con el siguiente comando:
```bash
$ sudo ufw delete allow ssh
```

## Instalación LEMP

### Paso 1: Instalar el servidor web Nginx

Para mostrar páginas web a los visitantes de nuestro sitio, emplearemos Nginx, un servidor web de alto rendimiento. Utilizaremos el administrador de paquetes apt para obtener este software.

Ya que esta es la primera vez que usamos apt para esta sesión, comience actualizando el índice de paquetes de su servidor. Después de eso, puede usar apt install para hacer instalar Nginx:

```bash
$ sudo apt update
$ sudo apt install nginx
```

Si tiene habilitado el firewall ufw, como se recomienda en nuestra guía de configuración inicial del servidor, deberá permitir las conexiones con Nginx. Nginx registra algunos perfiles de aplicaciones UFW diferentes al realizar la instalación. Para verificar qué perfiles UFW están disponibles, ejecute lo siguiente:

```bash
$ sudo ufw app list
```

```bash
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
```

Puede habilitarlo escribiendo lo siguiente:
```bash
$ sudo ufw allow 'Nginx HTTP'
$ sudo ufw status
```

### Paso 2: Instalar MariaDB
Ahora que su servidor web está listo, debe instalar un sistema de base de datos para poder almacenar y gestionar los datos de su sitio. MariaDB es un sistema de administración de bases de datos popular que se utiliza en entornos PHP.

```bash
$ sudo apt install mariadb mariadb-server
```

#### Configuración de mariadb por defecto
```bash
$ sudo mysql_secure_installation
```

#### Permisos de MariaDB

##### Cómo crear un nuevo usuario

Comencemos creando un nuevo usuario en el shell de MySQL:

```bash
mysql > CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
```
En este momento, newuser no tiene permisos para hacer nada con las bases de datos. De hecho, incluso si newuser intenta iniciar sesión (con la contraseña, password), no podrá acceder al shell de MySQL.

Por lo tanto, lo primero que se debe hacer es proporcionar al usuario acceso a la información que necesitará.

```bash
mysql > GRANT ALL PRIVILEGES ON * . * TO 'newuser'@'localhost';
```

Una vez que haya finalizado los permisos que desea configurar para sus nuevos usuarios, asegúrese siempre de volver a cargar todos los privilegios.
```bash
mysql > FLUSH PRIVILEGES;
```

##### Cómo otorgar diferentes permisos de usuario
Aquí se incluye una breve lista de otros posibles permisos comunes que los usuarios pueden disfrutar.

- ALL PRIVILEGES: Como vimos antes, esto le otorgaría a un usuario de MySQL acceso completo a una base de datos designada (o si no se selecciona ninguna base de datos, acceso global a todo el sistema).
- CREATE: Permite crear nuevas tablas o bases de datos.
- DROP: Permite eliminar tablas o bases de datos.
- DELETE: Permite eliminar filas de las tablas.
- INSERT: Permite insertar filas en las tablas.
- SELECT: Les permite usar el comando SELECT para leer las bases de datos.
- UPDATE: Permite actualizar las filas de las tablas.
- GRANT OPTION: Permite otorgar o eliminar privilegios de otros usuarios.

Para proporcionar un permiso a un usuario específico, puede usar este marco:
```bash
mysql > GRANT type_of_permission ON database_name.table_name TO 'username'@'localhost';
```

Si necesita revocar un permiso, la estructura es casi la misma que para otorgar un permiso:
```bash
mysql > REVOKE type_of_permission ON database_name.table_name FROM 'username'@'localhost';
```

Puede revisar los permisos actuales de un usuario ejecutando lo siguiente:
```bash
mysql > SHOW GRANTS FOR 'username'@'localhost';
```

Al igual que puede eliminar bases de datos con DROP, también puede usar DROP para eliminar un usuario por completo:
```bash
mysql > DROP USER 'username'@'localhost';
```

### Instalar PHP
Instaló Nginx para suministrar su contenido y MySQL para almacenar y administrar sus datos. Ahora puede instalar PHP a fin de procesar el código y generar contenido dinámico para el servidor web.

Aunque Apache integra el intérprete PHP en cada solicitud, Nginx requiere un programa externo para gestionar el procesamiento de PHP y actuar como puente entre el intérprete PHP en sí y el servidor web. Esto permite un mejor rendimiento general en la mayoría de los sitios web basados en PHP, pero requiere configuración adicional. Deberá instalar php-fpm, que significa “PHP fastCGI process manager”, e indicar a Nginx que pase solicitudes PHP a este software para su procesamiento. Además, necesitará php-mysql, un módulo PHP que permite a PHP comunicarse con las bases de datos basadas en MySQL. Los paquetes PHP básicos se instalarán automáticamente como dependencias.

#### Repositorios php
Link del repositorio: https://packages.sury.org/php/README.txt

```bash
$ sudo nala update
$ sudo nala install lsb-release ca-certificates curl
$ sudo curl -sSLo /tmp/debsuryorg-archive-keyring.deb https://packages.sury.org/debsuryorg-archive-keyring.deb
$ sudo dpkg -i /tmp/debsuryorg-archive-keyring.deb
$ sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
$ sudo nala update
```

```bash
$ sudo nala install php-cli php-fpm php-mysql php-zip php-gd php-mcrypt php-mbstring php-curl php-xml php-pear php-bcmath php-json
```

### Configurar Nginx para utilizar el procesador PHP
Al emplear el servidor web Nginx, podemos crear bloques de servidor (similares a los hosts virtuales de Apache) para encapsular los detalles de configuración y alojar más de un dominio en un único servidor. 

Cree el directorio web root para your_domain de la siguiente manera:
```bash
$ sudo mkdir /var/www/your_domain
```

A continuación, asigne la propiedad del directorio con la variable de entorno $USER, que hará referencia a su usuario de sistema actual:
```bash
$ sudo chown -R $USER:$USER /var/www/your_domain
```

Luego, abra un nuevo archivo de configuración en el directorio sites-available de Nginx con el editor de línea de comandos que prefiera. En este caso, utilizaremos nano:
```bash
$ sudo nano /etc/nginx/sites-available/your_domain
```

De esta manera, se creará un nuevo archivo en blanco. Pegue la siguiente configuración básica:
```bash
# /etc/nginx/sites-available/your_domain
server {
	    listen 8080;

	    root /var/www/test;

	    # Add index.php to the list if you are using PHP
	    index index.html index.php index.htm index.nginx-debian.html;

	    server_name _;

	    location / {
		    # First attempt to serve request as file, then
		    # as directory, then fall back to displaying a 404.
		    try_files $uri $uri/ =404;
	    }

	    location ~ \.php$ {
        	include snippets/fastcgi-php.conf;
        	fastcgi_pass unix:/run/php/php-fpm.sock;
     	}

}
```

Establezca un vínculo con archivo de configuración del directorio sites-enabled de Nginx para activar su configuración:
```bash
$ sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

Esto le indicará a Nginx que utilice la configuración la próxima vez que se vuelva a cargar. Puede verificar si hay errores en la sintaxis de su configuración al escribir:
```bash
$ sudo nginx -t
```

Cuando esté listo, vuelva a cargar Nginx para aplicar los cambios:
```bash
$ sudo systemctl reload nginx
```

Ahora, su nuevo sitio web está activo, pero el directorio root web /var/www/your_domain todavía está vacío. Cree un archivo index.html en esa ubicación para poder probar que el nuevo bloque del servidor funcione según lo previsto:
```bash
$ nano /var/www/your_domain/index.html
```
Incluya el siguiente contenido en este archivo:
```bash
<html>
  <head>
    <title>your_domain website</title>
  </head>
  <body>
    <h1>Hello World!</h1>

    <p>This is the landing page of <strong>your_domain</strong>.</p>
  </body>
</html>
```

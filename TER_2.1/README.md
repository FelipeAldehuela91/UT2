# Tarea Evaluable TER_2.1

## Hecho por Felipe Aldehuela Flete

### Paso 1.
1. Creamos la carpeta "UT2/TER_2.1"
![Paso 1](./img/Imagen%201.0.PNG)
1.1 Hacemos la estructura en árbol:
![Paso 1.1](./img/Imagen%201.1.PNG)
1.2 Nos descargamos los recursos necesarios y los metemos en la carpeta docker-lamp.
![Paso 1.2](./img/Imagen%201.2.PNG)

### Paso 2.
1. Creamos una imagen de Docker que incluye Apache y PHP, Y utilizaremos el siguiente comando:
~~~
docker run -ti --name daw_te2_1 php:8.0.0-apache /bin/bash
~~~
![Paso 2](./img/Imagen%202.0.PNG)

Explicación de lo que hemos usado:
- docker run: Crea contenedores de las imágenes.
- -ti: Con esto el contenedor es interactivo.
- --name daw_ter2_1: El nombre que le ponemos a nuestro contenedor.
- php:8.0.0-apache: Nombre de la imagen, y que usaremos la versión 8.0.0 que ya tiene apache instalado.
2. Luego de usar el comando podemos ver que estamos conectados al contenedor, y todo lo que hagamos será dentro de él.
![Paso 2.1](./img/Imagen%202.1.PNG)
3. Ahora vamos a instalar el driver necesario de MySQL con el siguiente comando:
~~~
docker-php-ext-install mysqli
~~~
![Paso 2.2](./img/Imagen%202.2.PNG)

Ya que así lo hará dentro del contenedor.
![Paso 2.3](./img/Imagen%202.0.gif)
4. Ahora vamos a instalar librerias del driver de MySQL, ya que son necesarios para que funcione correctamente, por eso vamos a usar varios comandos seguidos:
~~~
apt-get update

apt-get install -y sendmail libpng-dev 
apt-get install -y libzip-dev 
apt-get install -y zlib1g-dev 
apt-get install -y libonig-dev 
rm -rf /var/lib/apt/lists/* 
docker-php-ext-install zip

docker-php-ext-install mbstring
docker-php-ext-install zip
docker-php-ext-install gd
~~~
![Paso 2.4](./img/Imagen%202.3.PNG)
5. Por último usamos el comando:
~~~
a2enmod rewrite
~~~
![Paso 2.5](./img/Imagen%202.4.PNG)
![Paso 2.6](./img/Imagen%202.5.PNG)
6. Con todos estos pasos ya tenemos listo el contenedor. Ahora vamos a construir la imagen de ese contenedor, para ello ponemos <strong>exit</strong> y salimos del contenedor.

7. Ponemos el comando:
~~~
docker ps -a
~~~
Que nos va a mostrar todos los contenedores que están en ejecución o no lo estén.

### Paso 3.
1. Vamos a crear un contenedor de la imagen anterior con el siguiente comando.
~~~
docker commit 09e80f984948 daw_php_apache_mysql
~~~
- Con <strong>commit</strong> También introducimos el id del contenedor base y el nombre de la imagen que vamos a crear.
![Paso 3](./img/Imagen%203.0.PNG)

### Paso 4.
1. Ahora vamos a crear y a usar un archivo dockerfile, ya que ayuda y simplifica mucho las cosas, y dicho archivo lo vamos a crear en <strong>TER_2.1/src/</strong> usando el siguiente código:
~~~
FROM php:8.0.0-apache
 
RUN docker-php-ext-install mysqli
RUN apt-get update \
    && apt-get install -y sendmail libpng-dev \
    && apt-get install -y libzip-dev \
    && apt-get install -y zlib1g-dev \
    && apt-get install -y libonig-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-install zip

RUN docker-php-ext-install mbstring
RUN docker-php-ext-install zip
RUN docker-php-ext-install gd

RUN a2enmod rewrite
~~~
Es el mismo código de antes pero tiene dos comandos nuevos:
- FROM: Con este le decimos de dónde procede la imagen. 
- RUN: Con este ejecutamos el comando dentro de la imagen durante la creación.

Las capas de una imagen son cruciales, representando cambios en la imagen y compartiéndose entre contenedores. Menos capas significan mejor rendimiento. Con un Dockerfile, podemos reducir capas, como uniendo instalaciones de librerías MySQL en una sola línea de comando.

Ahora vamos a construit la imagen con el siguiente comando:
~~~
docker build -t php-apache8.0-sdf:1.0 .
~~~
- Con Build: Se construye la imagen y le estamos diciendo que vamos a usar Dockerfile con el . final del comando.
- Con -t: Le decimos que vamos a introducir una versión.
- php-apache8.0-sdf:1.0: Es el nombre de la imagen, y con ":" le estamos diciendo su versión.
![Paso 4.0](./img/Imagen%204.0.gif)
![Paso 4.0](./img/Imagen%204.0.PNG)
![Paso 4.2](./img/Imagen%204.1.PNG)
Para comprobar que se ha creado ejecutamos:
~~~
docker images
~~~
![Paso 4.2](./img/Imagen%204.2.PNG)
Ahora la vamos a borrar para que no interfiera con la práctica:
~~~
docker rmi 31aa62e7f590

docker images
~~~
![Paso 4.3](./img/Imagen%204.3.PNG)

### Paso 4.
1. Docker-compose

Para un servicio de varias imagenes y contenedores se utiliza <strong>docker-compose</strong>. Usa un archivo <strong>YML</strong> donde hay una lista de todas las instrucciones que vamos a incluir en el servicio.

Se divide en tres partes: `servicios, volumenes y redes.`

- www: Es la primera página que vamos a construir. Se va a basar en la anterior que hemos creado. Por lo que vamos a hacer una referencia al archivo <strong>dockerfile</strong> que hemos usado anteriormente.

El código que usaremos será:
~~~
www:
    build: .
    image: daw/lamp-apache-php8-sdf:1.0
    ports: 
        - "9000:80"
    volumes:
        - ./www:/var/www/html
    depends_on:
        - db
    networks:
        - lamp-network
~~~
Vamos a explicar el código que hemos usado:

- www: Es el nombre del servicio. Vamos a construir la base de todo nuestro servicio, ya que va a tener PHP, Apache y MySQL.
- build: Con esto definimos cómo vamos a contruir la imagen. Al tener el <strong>dockerfile</strong> anterior usamos "." para hacer referencia a él, ya que están en la misma carpeta.
- image: Para definir el nombre de la imagen que vamos a crear y su versión.
- ports: Aquí definimos los puertos de la imagen, en una lista de pares host-contenedor separados por ":". Por ejemplo, "9000:80" asigna el puerto 9000 del host al puerto 80 del contenedor.
- volumenes: Define carpetas compartidas. Algo muy importante en <strong>Docker</strong> es que los contenedores pueden tener una carpeta compartida en nuestro sistema.
- depends_on: Define dependencias entre contenedores.
- networks: Asigna la red de trabajo.

2.
- db: Con este servicio nos encargamos de gestionar la base de datos:
~~~
db:
    image: mysql:8.0
    container_name: lamp-mysql-sdf
    ports: 
        - "3399:3306"
    command: --default-authentication-plugin=mysql_native_password
    environment:
        MYSQL_DATABASE: dbname
        MYSQL_ROOT_PASSWORD: test 
        MYSQL_USER: lamp
        MYSQL_PASSWORD: lamp
    volumes:
        - ./dump:/docker-entrypoint-initdb.d
        - ./conf:/etc/mysql/conf.d
        - db_data:/var/lib/mysql
    networks:
        - lamp-network
~~~

- db: Nombre del servicio.
- container_name: Nombre del contenedor que creamos.
- ports: Aquí se configuran los puertos de la imagen en una lista de pares host-contenedor separados por ":". Por ejemplo, "9000:80" asigna el puerto 9000 del host al puerto 80 del contenedor.
- environment: Aquí se declaran las variables de entorno del contenedor, como claves de conexión a la base de datos, contraseñas y/o usuarios.
- volumes: Aquí definimos la carpeta o carpetas compartidas del sistema para el contenedor. Cada una se especifica en una lista, similar a los puertos.
- networks: 
Aquí se asigna la red de trabajo al contenedor, que puede ser una red predeterminada de Docker o una personalizada.

3. 
- phpmyadmin:

Este último servicio creará un contenedor que servirá PHPMyAdmin para conectar a la base de datos a través del navegador web. Se conectará al contenedor de la base de datos para acceder a la información.
~~~
  phpmyadmin:
  image: phpmyadmin/phpmyadmin
  container_name: lamp-phpmyadmin-sdf
  depends_on: 
      - db
  ports:
      - 8900:80
  environment:
      MYSQL_USER: lamp
      MYSQL_PASSWORD: lamp
      MYSQL_ROOT_PASSWORD: test 
  networks:
      - lamp-network
~~~
- phpmyadmin: Nombre del servicio.
- image: Declarar la imagen base que vamos a usar.
- container_name: Nombre del contenedor que creamos.
- depends_on: Aquí se puede definir la dependencia o relación entre contenedores.
- ports: Aquí se especifican los puertos que utilizará la imagen. Cada puerto se define en una lista, indicando primero el puerto del host y luego el del contenedor, separados por ":". Por ejemplo, para asignar el puerto 9000 del host al puerto 80 del contenedor, se escribiría "9000:80".
- environment: Aquí se declaran las variables de entorno del contenedor. Por ejemplo, en este contexto, se pueden definir claves de conexión a la base de datos, contraseñas y/o usuarios.
- networks: Aquí se asignará la red con la que el contenedor trabajará. Puede ser una red predefinida de Docker o una red personalizada.

4.
- Volumenes:
Un contenedor es un entorno aislado que ejecuta lo que tengamos definido. Sus datos se pierden al eliminarlo, pero si deseamos conservar y trabajar con esos datos de forma recurrente, necesitamos una herramienta como Docker volumen.
~~~
volumes:
    db_data: 
        driver: local
~~~

5.
- Redes: Hay tres tipos de redes:
- Bridge: El puente (Bridge) es la red estándar y el controlador de red por defecto en Docker. Se crea automáticamente al iniciar la plataforma Docker y los contenedores se conectan a él, a menos que el usuario especifique lo contrario. Se utiliza comúnmente cuando las aplicaciones del cliente se ejecutan en contenedores independientes que necesitan comunicarse entre sí.
- Host: El controlador de redes en Docker conocido como "Host" elimina el aislamiento entre un contenedor y el host, lo que puede mejorar el rendimiento al eliminar la necesidad de traducción de direcciones de red y la creación de proxies de usuario para cada puerto. Este controlador es útil cuando un contenedor necesita controlar múltiples puertos.
- none: Esta opción de red de contenedores es la encargada de inhabilitar todas las redes de la plataforma de contenedores.

~~~
networks:
    lamp-network:
        driver: bridge
~~~

### Paso 5.
Vamos a probar que todo esto funciona con el siguiente comando:
~~~
docker-compose up --build
~~~
![Paso 5](./img/Imagen%205.0.gif)
Comprobamos que podemos entrar en localhost con:
`http://localhost:8900/`
![Paso 5.1](./img/Imagen%206.1.gif)
Ahora vamos a comprobar que con el Workbench también tiene que funcionar
![Paso 5.2](./img/Imagen%206.2.gif)

### Paso 6.
1. Parar el entorno
Para parar el entorno, ejecutamos el siguiente comando:
~~~
docker-compose down
~~~
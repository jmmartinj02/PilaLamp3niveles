# Configuración del Entorno LAMP en AWS

## Índice
1. [Creación de VPC](#creación-de-vpc)
2. [Creación de Instancias](#creación-de-instancias)
3. [Configuración de Instancias](#configuración-de-instancias)
   
    a. [Configuración instancias Apache](#configuración-instancias-apache)
   
    b. [Configuración instancias Apache](#configuración-instancias-apache)
   
    c. [Configuración instancias Apache](#configuración-instancias-apache)
5. [Obtención de Certificado SSL con Let's Encrypt](#obtención-de-certificado-ssl-con-lets-encrypt)
6. [Configuración de Seguridad](#configuración-de-seguridad)


## Creación de VPC

Para comenzar, crearemos una VPC para agrupar y aislar nuestra configuración. La configuración de las subredes tiene las siguientes características:

>![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/25bbbdbc-2c8d-4f4d-85ff-989eafdcbfa3)

**Así se vería en la vista previa de AWS:**

>![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/ab670d09-5058-4517-af1e-d03601832930)

- Todas las subredes comparten el mismo bloque CIDR para permitir la comunicación privada entre ellas.
- Se crea una quinta instancia puente para actuar como puente entre las instancias y la VPC.
- Se genera una IP elástica para que el balanceador de carga sea accesible desde el exterior, pero eso lo veremos más adelante.

## Creación de Instancias

El proceso de creación de instancias implica descargar las claves de conexión, configurar la red en la VPC correspondiente y asegurarse de que las instancias estén en la subred adecuada.

>**Este es el ejemplo de la máquina balanceador.**
>
>![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/ea4b02a0-bb3d-4df2-a0d9-884f76e70a0e)

Repetimos el proceso con las demás instacias, pero utilizando la subred privada en ellas.

>**Nuestro menú de instancias quedaría así:**
>
>![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/168944fc-96af-4e6e-939c-1765ef47cdc6)

**Pasos comunes para todas las instancias:**
- Descargar claves para la conexión SSH.
- Configurar la red asignándola a la VPC y subred correspondientes.
## Configuración de Instancias

**Para poder configurar nuestras instancias es necesario enviar por SCP la clave a la máquina puente.**

    scp -i balanceador.pem balanceador.pem admin@35.175.231.20:/$HOME

>![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/36e8b438-e21f-40eb-97d8-676a9185ce73)


### Configuración instancias Apache:
- Instalar Apache y los módulos de PHP necesarios.

      sudo apt install -y apache2 && sudo apt install php libapach2-mod-php php-mysql
  >**![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/ed86c45b-c7d9-4492-9003-ef5a8ba58488)

- Clonar la aplicación desde un repositorio Git y configurar el archivo de virtual host para Apache.
  
      sudo cp 000-default.conf practica2.conf
  
      sudo nano practica2.conf
  
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/93dc2bcd-a824-42e9-969c-da29cd26b5c1)


- Editamos el archivo añadiendo a DocumentRoot el nombre de la carpeta que crearemos mas adelante y src.

  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/2ecdc7ea-89eb-4ca5-b8f6-0307395b02f6)

- Damos de baja la configuración por defecto y habilitamos nuestro archivo en apache.

       sudo a2ensite practica2.conf
  
      sudo a2dissite 000-default.conf
  
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/915a0dec-ea0a-480f-8440-a8df996020bf)
  >
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/6c5488a1-a8b5-4133-b996-5dfbcae43d23)

- Instalamos git para poder clonar los archivos de la aplicación.

      sudo apt install -y git
      sudo mkdir /var/www/hmtl/practica2
      sudo git clone https://github.com/josejuansanchez/iaw-practica-lamp.git /var/www/html/practica2
  >
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/a0ee7e2c-aa73-4603-bbf3-8bacaf7bc359)
  >
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/4cb3d56e-034e-4df9-b769-4e7fc244d5ce)
  >
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/43cad3bf-cca7-494d-b3fb-5fdc5d437738)

- Editamos el archivo config.php que se encuentra en el interior de src.

  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/026c6e4c-b5c2-4b70-9780-09ba3c16a55b)

- Asegurarse de que Apache tenga acceso a los archivos.

      sudo git clone https://github.com/josejuansanchez/iaw-practica-lamp.git /var/www/html/practica2**
  
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/05f4b123-553a-4302-952d-427c9acd53d7)

- Reiniciamos Apache para que los cambios surtan efecto.

      sudo systemctl restart apache2**
  
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/80c81e98-106e-4333-afc9-8fc601189f4b)

- Transferimos vía SCP el archivo database.sql desde apache1 a la instancia puente, y desde la instancia puente, hacia la instancia datos

  >**Desde apache a puente**
  
      scp -i balanceador.pem admin@10.0.1.136:/var/www/html/practica2/db/database.sql ~/database.sql**
  
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/afe794c3-ebb5-4b62-8285-69fd06f74ccf)
  
  >**Desde puente a datos**
  >
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/f4d8f081-6c57-4208-a98c-fa1e1b8f4899)

-Repetimos estos pasos en la otra instacia apache, menos este último de transferencia del archivo.
  
## Pasos configuración instancia Datos (MariaDB):

- Instalar Mariadb-server.

      sudo apt install -y mariadb-server
  
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/3127509b-6c5b-404e-bb4c-40d2359ccf38)
  >
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/03a8a4d5-4554-412f-b0b6-49e04f23126d)

- Editamos el archivo 50-server.conf que se encuentra en /etc/mysql/mariadb.conf.d

       sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
  
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/bd0aa7cc-f129-4479-bdb8-097c604ea72c)

- configurar la base datos que hemos importado con SCP, crear el usuario y darle privilegios.

      sudo mysql -u root < $HOME/database.sql
      sudo mysql -u root
  
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/d60ce8a3-cc62-42be-909b-2de7f27e9455)
  
      CREATE USER 'usuarios_user@10.0.10.%' IDENTIFIED BY '1234';
      GRANT ALL PRIVILEGES ON lamp_db.* TO 'usuarios_user@10.0.10.%';
      FLUSH PRIVILEGES;
  
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/aec6cf2f-5aa1-43bb-9ffc-620db5be9da8)


## Configuración Instancia Balanceador.

- Para que podamos acceder a la aplicación desde el exterior necesitaremos asociar una IP elástica a la instancia balanceador.

  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/170be8af-ac9a-48df-a3a8-08ac3a4f155d)

  La asociamos con la instancia, en red y seguridad, seleccionamos IP elásticas y teniendo marcada la IP la asociamos desde el menú acciones.
  Debemos asegurarnos de que la instancia es la del balanceador y introducir también la ip privada de dicha instancia.

  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/4c264f4b-99bc-489e-9b3d-ed2b5d01baf8)


- Instalar Apache2 en la instancia balanceador.

      sudo apt install -y apache2

  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/831d7540-e3a3-4641-962b-42ebbf849bbe)

- Habilitar los módulos necesarios para el balanceador de carga.

      sudo a2enmod proxy_ajp
      sudo a2enmod rewrite
      sudo a2enmod deflate
      sudo a2enmod headers
      sudo a2enmod proxy_balancer
      sudo a2enmod proxy_connect
      sudo a2enmod proxy_html
      sudo a2enmod lbmethod_byrequests
      sudo a2enmod proxy
      sudo a2enmod proxy_http
      sudo a2enmod ssl

    >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/a4b69990-cc9e-43c9-ab4b-9cdb728e4bb1)


- Configurar el archivo de virtual host del balanceador con las direcciones IP privadas de las instancias Apache.
  
      etc/apache2/sites-available  
      sudo cp default-ssl.conf balanceador.conf
      sudo a2ensite balanceador.conf
      sudo a2dissite 000-default.cof

  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/deccc2ec-2d55-4559-860c-a00d9d46b011)

  
      sudo systemctl restart apache2
      
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/4cc95c98-c8bb-4a18-b048-191116c2c131)


## Obtención de Certificado SSL con Let's Encrypt
- Crear un dominio y asignarle una ip elástica utilizando NO-IP
  >
  >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/73872692-d6c4-4a75-b9ed-b2aee3be7e8b)

- Utilizar Certbot para obtener un certificado SSL.
    Instalamos snapd:
  
      sudo apt install snapd

    >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/2fa5ebb0-8ecf-4054-9dc7-4abf45992c90)

    Instalamos snap core:

      sudo snap install core

    >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/d58c43bc-0877-4e82-9e00-ca23822cde6e)

    Instalamos certbot:

      sudo snap install --classic certbot

    >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/9c820ca1-5a8a-4589-bb31-bfaac85eca64)

    Lanzamos certbot:

      sudo certbot --apache

    >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/803aed27-b17b-49c3-9634-ed21235a0752)

    Pedirá un correo electrónico, lo introducimos.
    Si queremos aceptar los términos y servicios, decimos que sí.
    Si queremos compartir el correo para recibir información adicional, diremos que no.
    Finalmente nos pedirá el nombre de nuestro dominio, en mi caso es jmmartinj.ddns.net

    >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/cafb1f75-0f4f-457a-ab03-716cfc884136)


## Configuración de Seguridad

- Hay que asegurarse de que las instancias solo permitan conexiones según los requisitos de la aplicación.
  - Balanceador: HTTPS (puerto 443) desde internet y HTTP (puerto 80) hacia instancias Apache.

    >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/b5550c2c-807a-4984-adec-a8a86384111f)

  - Instancias Apache: HTTP (puerto 80) desde el balanceador y MySQL (puerto 3306) desde la instancia Datos.

    >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/d33635f6-bc0f-4eb1-bbc0-e3bb83bec624)

  - Instancia Datos: Dos entradas MySQL (puerto 3306), una desde cada una de las instancias Apache.

    >![image](https://github.com/jmmartinj02/PilaLamp3niveles/assets/146434706/54fdff06-9693-427a-a8df-48e2782ee642)

Este es un resumen de los pasos y configuraciones realizados en el entorno LAMP en AWS. Tenga en cuenta que quizá debas adaptar un poco su configuración para que esté a su gusto.

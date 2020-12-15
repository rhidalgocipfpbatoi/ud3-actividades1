# Acceso a Base de Datos. PDO (PHP Data Objects)

## Preparando el entorno

### 1. Importación de la base de datos
 El primer paso para la preparación del entorno es la importación del ``schema`` de la base de dades contenida en el [siguiente](/database/crm_db.sql) script.  
La opción más fácil es volcar el contenido del fichero mediante el cliente de línea de comandos ``mysql client``, y utilizando el usuario por defecto del sistema.

```bash
    sudo mysql -u'root' < database/crm_db.sql 
```

~~~
    La instalación por defecto del SGBD mysql en distribuciones "Ubuntu Server ~18.04" proporciona un usuario "root" sin "contraseña" con grants "ALL_PRIVILEGES"
    y que puede establecer solo conexiones locales (desde el mismo host) a través del socket LINUX situado en "/var/run/mysqld/mysqld.sock"
~~~

### 2. Creación del usuario de conexión
Una vez importada la base de datos crearemos un usuario para nuestra aplicación ``php`` debemos tener en cuenta que:
- Debe ser un usuario con los mínimos privilegios que requiera nuestra aplicación para que, en caso que existiera alguna vulnerabilidad,
 el daño fuera mínimo.
- No debemos compartir usuarios con distintas aplicaciones, en otro caso, estariamos introduciendo acoplamientos y vulnerabilidades
entre aplicaciones.
- El usuario que creemos solo tendrá permisos de acceso desde el host o la ip en la que se vaya a ejecutar nuestra aplicación. En nuestro
caso ``localhost``.

```bash
  
    CREATE USER descriptive-user-name@'ip-from-connection-host' IDENTIFIED BY 'strong-password';
    
    #Creamos el usuario
    CREATE USER 'alecogi-web'@'%' IDENTIFIED BY '123456789'
    
    #Le concedemos todos los privileios para la base de datos creada
    GRANT ALL PRIVILEGES ON crm_db.* TO 'alecogi-web'@'%'
    
```

El [siguiente enlace](https://www.digitalocean.com/community/tutorials/como-instalar-mysql-en-ubuntu-18-04-es) nos propociona un resumen sobre una instalación básica del servidor **mysql** en **Ubuntu Server 18.04**

~~~
Debemos tener en cuenta que en algunas distribuciones mysql se encuentra configurado por defecto para atender solamente peticiones
del mismos host **localhost** si queremos conectarnos desde otro host deberemos bindear el servicio a la interfaz de red correspondiente.

$sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address            = 0.0.0.0
~~~

### 3. Instalación del driver nativo php-mysql

Como último paso llevaremos acabo la instalación del driver de mysql para php
```bash
$sudo apt install php-mysql
```
**Nota** no olvides reinicar el servidor php

## Actividad 1

**A)** Crea un directorio ``resources`` y crea script ``database-params.php`` en el que guardaremos los parámetros de acceso a la base de datos.

```php
<?php
   $dataBaseParams = [
          "host" => ... ,
          "user" => ... ,
          "password" => ... ,
          "database" => ...
   ]; 
```
**B)** Crea un script ``connection.php`` en el que:
  - A partir de la inclusión del fichero anterior cree una conexión con la BD.
  - Crea una segundo script ``connection_error.php``. En este caso debe utilizar unas credenciales erróneas.
    Muestra por pantalla el mensaje y código ``SQLSTATE`` correspondiente.
 
```php
<?php   
    
    ini_set('display_errors', 1);
    error_reporting(E_ALL);
    
    require __DIR__ . "/../resources/database-params.php";
    
     try {
    
         // Accedemos a cada uno de los parámetros de acceso a la base de datos
         $usuario = $dataBaseParams['user'];
         $contraseña =  $dataBaseParams['password'];
         $ipServidor = $dataBaseParams['host'];
         $databaseName = $dataBaseParams['database'];
         
         //Creamos la conexión con la base de datos a partir de los parámetros leidos
         $conexión = new PDO("mysql:host=$ipServidor;dbname=$databaseName", $usuario, $contraseña);
         echo "Conexion establecida ";
    
        } catch (PDOException $e) {
   
            echo "Error en la conexión con la Base de datos " . $e->getMessage() . "<br/>";
            die();
    
        }
```
~~~
Todos los scripts que creemos incluiran el fichero ``database-params.php`` para obtener los párametros de conexión
con la base de datos
~~~

**C)** Crea un script ``listado_usuarios.php`` que lea y muestre todos los usuarios de la base de datos. Debe mostrarlos
 en una elemento ``<html>``

```php
<?php  

    //incluimos el fichero creado en el apartado anterior de forma que ya disponemos de una conexión a la BD
    include __DIR__. "/connection.php";

    //Creamos el select de acceso a la base de datos
    $sql = "SELECT id,firstName, lastName, phoneNumber, active, createdOn FROM User";

    //Ejecutamos la sentencia
    $listadoUsuarios = $conexión->query($sql);
    
    //Iteramos a través del array de resultados
    foreach ($listadoUsuarios as $row) {
        
        //Accedemos a cada uno de los elementos
        echo($row['firstName']);
        echo($row['lastName']);
        
    }
    
```

**D)** Crea un script ``listado_empresas.php`` que lea y muestre todas las empresas de la base de datos. Al igual que el 
anterior debe mostrarlo en un elemento ```<table>```.

**E)** Crea un script ``inserta_empresas.php`` Que dé de alta las siguientes empresas

**Nota:** Deberás tener en cuenta el oden de inserción.

**Empresas**

| name | address | city | province | country | locale | nif | status | 
|---|---|---|---|---|---|---|---|
| Empresa1 | C/ Alicante, 1 | Alcoi | Alicante | ES | es_ES | 21543341R | 1 |
| Empresa2 | C/ Alicante, 2 | Cocentaina | Alicante | ES | es_ES | 11543341R | 1 |
| Empresa3 | C/ La Rambla, 3 | Ontinyent | Valencia | ES | es_ES | 41543341R | 1 |

**F)** Crea un script ``inserta_usuarios.php`` Que dé de alta las siguientes usuarios

**Usuarios**

| firstName | lastName  | email  | phoneNumber | password | locale | Enterprise | birthday |
|---|---|---|---|---|---|---|---|
| Empar | Seguí Domenech | cliente1@empresa1.com  | cambiame | 111111111 | ca_ES | Empresa1 | 1990-01-08 |
| Vanesa | García Perez  | cliente2@empresa1.com  |  cambiame | 222222222  | es_ES | Empresa1 | 1984-08-02 | 
| Monica  | Fernandez Fernandez | cliente3@empresa1.com  | cambiame | 333333333 | en_GB | Empresa1 | 1980-08-02 |
| Elena  | Martín Sanchez  | cliente4@empresa2.com  | cambiame  | 444444444 | en_US | Empresa1 | 1994-08-02 |

**G)** Crea un script ``baja_usuarios.php`` que desactive todos los usuarios de la Empresa1.

**H)** Crea un script ``elimina_usuarios.php`` que elimine los usuarios cuyo email termine en ``test.com``

**I)** Crea un script ``dml_usuarios_empresas.php`` que deberá llevar a cabo las siguientes acciones.

- Crear una tabla html `<table>` con los 5 primeros usuarios de la `Empresa1` ordenados por el campo id.
- Mostrar una tabla html `<table>` con todas las empresas de la provincia de Alicante.
- Insertar 10 usuarios generados de forma aleatoria a partir de la combinación de datos presentes en el siguiente array.

```
<?php
    
    $informacionBase = [
        "firstName" => ["Rafa", "Alejandro", "Bea", "Joaquin", "Oscar", "Alvaro", "Sergio", "Matias"],
        "lastName" => ["Máximo","Cervera","Finalet","Herrero","Gilaber","Avellán","Llorca","Lucas","Millan"],
        "locale" => ["ca_ES", "es_ES", "en_GB"]
    ]
```

- El `email` vendrà dado de la concatenación de los 2 primeros caracteres del nombre y apellidos seguidos de
 `@{nombre_de_la_empresa}.com` 
- El `password` será un hash de la palabra `cambiame` generado a partir del uso de la función [password_hash()](https://www.php.net/manual/es/function.password-hash.php)
- El teléfono vendrá dado por un número aleatorio entre `4000` y `5000`.
- La fecha de nacimiento será un día aleatorio del mes de noviembre de 1981. `1981-11-{dia}-11`.
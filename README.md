# ALUMNO 1

### 1. Realiza una exportación del esquema de SCOTT usando la consola Enterprise Manager, con las siguientes condiciones:

    • Exporta tanto la estructura de las tablas como los datos de las mismas.
    • Excluye la tabla BONUS y los departamentos sin empleados.
    • Realiza una estimación previa del tamaño necesario para el fichero de exportación.
    • Programa la operación para dentro de 5 minutos.
    • Genera un archivo de log en el directorio raíz.
      
Realiza ahora la operación con Oracle Data Pump.


- ORACLE DATA PUMP

1.

Creamos un directorio en la máquina:

oracle@OracleJessie:~$ mkdir scott

Creamos un directorio dentro de la base de datos:

SQL> CREATE DIRECTORY delegado as '/home/oracle/scott';

Directorio creado.

Damos permiso a system:

SQL> GRANT EXP_FULL_DATABASE TO system;

Concesión terminada correctamente.

SQL> GRANT IMP_FULL_DATABASE TO system;

Concesión terminada correctamente.

SQL> GRANT READ, WRITE ON DIRECTORY delegado TO system;

Concesión terminada correctamente.

Realizamos la exportación:

oracle@OracleJessie:~$ expdp system/RAUL schemas=SCOTT directory=delegado dumpfile=copiascott.dmp logfile=scottesquema.log

Listamos el directorio y vemos ambos ficheros:

oracle@OracleJessie:~/scott$ ls
copiascott.dmp	scottesquema.log

### 2. Importa el fichero obtenido anteriormente usando Enterprise Manager pero en un usuario distinto de la misma base de datos.

### 3. Realiza una exportación de la estructura de todas las tablas de la base de datos usando el comando expdp de Oracle Data Pump probando todas las posibles opciones que ofrece dicho comando y documentándolas adecuadamente.

### 4. Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con MySQL desde línea de comandos, documentando el proceso.

Para exportar en MariaDB desde línea de comandos usamos 'mysqldump'. Indicamos el nombre de usuario (-u), el nombre de la base de datos (-p) y el nombre de la copia exportada tras el símbolo '>':

~~~
debian@pruebas2:~$ mysqldump -u ftirado -p universidad > copiauniversidad.sql
Enter password: 
~~~

Listamos y vemos que se ha generado correctamente:

~~~
debian@pruebas2:~$ ls
copiauniversidad.sql
~~~

Verificamos las 5 primeras líneas del fichero, donde se indican diferentes datos de la copia:

~~~
debian@pruebas2:~$ head -n 5 copiauniversidad.sql 
-- MySQL dump 10.17  Distrib 10.3.22-MariaDB, for debian-linux-gnu (x86_64)
--
-- Host: localhost    Database: universidad
-- ------------------------------------------------------
-- Server version	10.3.22-MariaDB-0+deb10u1
~~~

Tras esto, vamos a realizar la importación. Lo primero que vamos a hacer es crear una nueva base de datos con el usuario root:

~~~
MariaDB [(none)]> create database nuevauniversidad;
Query OK, 1 row affected (0.001 sec)
~~~

Realizamos la importación:

~~~
debian@pruebas2:~$ mysql -u delegado -p nuevauniversidad < copiauniversidad.sql
Enter password:
~~~

~~~
MariaDB [nuevauniversidad]> show tables;
+----------------------------+
| Tables_in_nuevauniversidad |
+----------------------------+
| Personal                   |
+----------------------------+
1 row in set (0.001 sec)
~~~

### 5. Realiza operaciones de importación y exportación de colecciones en MongoDB.

Vamos a crear una base de datos de ejemplo:

~~~
> use delegado
switched to db delegado
~~~

Creamos también una colección de ejemplo y listamos:

~~~
> db.delegado.insertOne( {
...     usuario_id: "abc123",
...     nombre: "Fernando",
...     edad: 24,
...     categoria: "A"
...  } )
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5e5e9fe2967697ce195bea13")
}
> db.delegado.find()
{ "_id" : ObjectId("5e5e9fe2967697ce195bea13"), "usuario_id" : "abc123", "nombre" : "Fernando", "edad" : 24, "categoria" : "A" }
~~~

Ahora vamos a realizar una exportación de este ejemplo y vemos el contenido:

~~~
vagrant@mongo:~$ mongoexport --collection=delegado --db=delegado --out=delegado.json
2020-03-03T18:23:30.863+0000	connected to: mongodb://localhost/
2020-03-03T18:23:30.871+0000	exported 1 record
vagrant@mongo:~$ ls
delegado.json
vagrant@mongo:~$ cat delegado.json 
{"_id":{"$oid":"5e5e9fe2967697ce195bea13"},"usuario_id":"abc123","nombre":"Fernando","edad":24.0,"categoria":"A"}
~~~

Finalmente vamos a realizar una importación usando el fichero json creado al exportar:

~~~
vagrant@mongo:~$ mongoimport --db=espias --collection=agentes --file=delegado.json
2020-03-03T18:26:43.560+0000	connected to: mongodb://localhost/
2020-03-03T18:26:43.571+0000	1 document(s) imported successfully. 0 document(s) failed to import.
~~~

Comprobamos los datos importados:

~~~
> use espias
switched to db espias
> db.agentes.find()
{ "_id" : ObjectId("5e5e9fe2967697ce195bea13"), "usuario_id" : "abc123", "nombre" : "Fernando", "edad" : 24, "categoria" : "A" }
~~~

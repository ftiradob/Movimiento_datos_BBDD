# ALUMNO 1

### 1. Realiza una exportación del esquema de SCOTT usando la consola Enterprise Manager, con las siguientes condiciones:

    • Exporta tanto la estructura de las tablas como los datos de las mismas.
    • Excluye la tabla BONUS y los departamentos sin empleados.
    • Realiza una estimación previa del tamaño necesario para el fichero de exportación.
    • Programa la operación para dentro de 5 minutos.
    • Genera un archivo de log en el directorio raíz.
      
Realiza ahora la operación con Oracle Data Pump.


- ORACLE DATA PUMP

Creamos un directorio en la máquina:

~~~
oracle@OracleJessie:~$ mkdir scott
~~~

Creamos un directorio dentro de la base de datos:

~~~
SQL> CREATE DIRECTORY delegado as '/home/oracle/scott';

Directorio creado.
~~~

Damos permiso a system:

~~~
SQL> GRANT EXP_FULL_DATABASE TO system;

Concesión terminada correctamente.

SQL> GRANT IMP_FULL_DATABASE TO system;

Concesión terminada correctamente.

SQL> GRANT READ, WRITE ON DIRECTORY delegado TO system;

Concesión terminada correctamente.
~~~

Antes de realizar la exportación vamos a hacer una estimación:

~~~
oracle@OracleJessie:~$ expdp system/RAUL schemas=SCOTT ESTIMATE_ONLY=YES directory=delegado

Export: Release 12.1.0.2.0 - Production on Mar Mar 3 20:53:29 2020

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

Conectado a: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

Advertencia: las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raíz o al elemento inicial de una base de datos del contenedor.

Iniciando "SYSTEM"."SYS_EXPORT_SCHEMA_01":  system/******** schemas=SCOTT ESTIMATE_ONLY=YES directory=delegado 
Estimación en curso mediante el método BLOCKS...
Estimación total mediante el método BLOCKS: 0 KB
El trabajo "SYSTEM"."SYS_EXPORT_SCHEMA_01" ha terminado correctamente en Mar Mar 3 20:53:33 2020 elapsed 0 00:00:03
~~~

Es normal que nos indique 0 KB ya que el esquema SCOTT solamente tiene 2 tablas y apenas ocupa espacio.

Realizamos la exportación del esquema SCOTT, indicando que además cree un archivo de log:

~~~
oracle@OracleJessie:~$ expdp system/RAUL schemas=SCOTT directory=delegado dumpfile=copiascott.dmp logfile=scottesquema.log
~~~

Listamos el directorio y vemos ambos ficheros:

~~~
oracle@OracleJessie:~/scott$ ls
copiascott.dmp	scottesquema.log
~~~

Para programar la operación dentro de 5 minutos vamos a realizar una tarea de cron. Primero realizamos el siguiente script:

~~~
oracle@OracleJessie:~$ sudo nano script5minutos.sh
#!/bin/bash
expdp system/RAUL schemas=SCOTT directory=delegado dumpfile=copiascott.dmp logfile=scottesquema.log
~~~

Le cambiamos los permisos:

~~~
oracle@OracleJessie:~$ sudo chmod 744 script5minutos.sh
~~~

Creamos la tarea de cron:

~~~
oracle@OracleJessie:~$ sudo crontab -e
*/5 * * * * /home/oracle/script5minutos.sh
~~~

Reiniciamos el servicio y ya tendríamos nuestro cron funcionando.

### 2. Importa el fichero obtenido anteriormente usando Enterprise Manager pero en un usuario distinto de la misma base de datos.

### 3. Realiza una exportación de la estructura de todas las tablas de la base de datos usando el comando expdp de Oracle Data Pump probando todas las posibles opciones que ofrece dicho comando y documentándolas adecuadamente.

Lo primero que tenemos que hacer es crear un directorio en la base de datos vinculándolo con un directorio de la máquina:

~~~
SQL> CREATE DIRECTORY datosdelegado as '/home/oracle/datosdelegado';

Directorio creado.
~~~

Ahora tenemos que darle los permisos necesarios al usuario 'system' para realizar esta tarea:

~~~
SQL> GRANT EXP_FULL_DATABASE to system;

Concesión terminada correctamente.

SQL> GRANT IMP_FULL_DATABASE to system;

Concesión terminada correctamente.

SQL> GRANT READ, WRITE ON DIRECTORY datosdelegado TO system;

Concesión terminada correctamente.
~~~

Ahora vamos a realizar la exportación:

~~~
oracle@OracleJessie:~$ expdp system/RAUL directory=datosdelegado dumpfile=datosdelegado.dmp logfile=logdelegado.log

Export: Release 12.1.0.2.0 - Production on Mar Mar 3 20:44:42 2020

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

Conectado a: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

Advertencia: las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raíz o al elemento inicial de una base de datos del contenedor.

Iniciando "SYSTEM"."SYS_EXPORT_SCHEMA_01":  system/******** directory=datosdelegado dumpfile=datosdelegado.dmp logfile=logdelegado.log 
Estimación en curso mediante el método BLOCKS...
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
Estimación total mediante el método BLOCKS: 0 KB
Procesando el tipo de objeto SCHEMA_EXPORT/DEFAULT_ROLE
Procesando el tipo de objeto SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/COMMENT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/INDEX
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/STATISTICS/MARKER
Procesando el tipo de objeto SCHEMA_EXPORT/POST_SCHEMA/PROCACT_SCHEMA
La tabla maestra "SYSTEM"."SYS_EXPORT_SCHEMA_01" se ha cargado/descargado correctamente
******************************************************************************
El juego de archivos de volcado para SYSTEM.SYS_EXPORT_SCHEMA_01 es:
  /home/oracle/datosdelegado/datosdelegado.dmp
El trabajo "SYSTEM"."SYS_EXPORT_SCHEMA_01" ha terminado correctamente en Mar Mar 3 20:45:40 2020 elapsed 0 00:00:55
~~~

Las opciones más importantes que ofrece Oracle Data Pump son:

- COMPRESSION => Especifica si la exportación va a ser comprimida.
- CONTENT => Filtrar lo que exportas.
- DIRECTORY => Especificamos la localización en la que la exportación puede excribir.
- DUMPFILE => Especifica el nombre del fichero exportado.
- ENCRYPTION_PASSWORD => Especifica una clave para la encriptación.
- ESTIMATE => Usado para realizar una estimación en la exportación.
- ESTIMATE_ONLY => Solamente realiza una estimación.
- EXCLUDE => Excluye el objeto que le indiquemos.
- FILESIZE => Especificamos el tamaño máximo del archivo exportado.
- FULL => Especifica que exporte todo: estructura de tablas, datos y metadatos.
- LOGFILE => Especifica el nombre del archivo de log.
- NOLOGFILE => Especifica que no cree fichero de log.
- SCHEMAS => Especifica un esquema concreto.
- TABLES => Especifica una tabla concreta.


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

Ahora vamos a realizar una exportación de este ejemplo. Usando la herramienta 'mongoexport' indicándole la colección que queremos exportar (--collection), la base de datos en la que esta (--db) y el nombre del fichero json que se va a generar (--out):

~~~
vagrant@mongo:~$ mongoexport --collection=delegado --db=delegado --out=delegado.json
2020-03-03T18:23:30.863+0000	connected to: mongodb://localhost/
2020-03-03T18:23:30.871+0000	exported 1 record
~~~

Vemos el contenido:

~~~
vagrant@mongo:~$ ls
delegado.json
vagrant@mongo:~$ cat delegado.json 
{"_id":{"$oid":"5e5e9fe2967697ce195bea13"},"usuario_id":"abc123","nombre":"Fernando","edad":24.0,"categoria":"A"}
~~~

Finalmente vamos a realizar una importación usando el fichero json creado al exportar. Usando la herramienta 'mongoimport' indicándole la base de datos donde queremos importarla (--db), el nombre que tendrá la colección (--collection) y el nombre del fichero json que exportamos anteriormente (--file):

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

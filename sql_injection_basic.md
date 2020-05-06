# INYECCIÓN SQLi - Básico 

## TEMAS

* [¿Qué es una inyección sql?](#¿Qué-es-una-inyección-sql?)
* [¿Porqué ocurre un error sql?](#¿Porqué-ocurre-un-error-sql?)
* [Tipos de inyección sql](#Tipos-de-inyección-sql)
    * [Inyección sql manual](#Inyección-sql-manual)
      * [Detectar una página vulnerable](#Detectar-una-página-vulnerable)
    * [Inyección sql automatizada con sqlmap](#Inyección-sql-automatizada-con-sqlmap)


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

``
id=1
id2=1
id3=1
``

Ahora, para detectar si la página es vulnerable, una de las cosas mas sencillas es usar un simple ' al final de cada para metro para ver si nos genera un error en la base de datos.

## Inyección sql automatizada con sqlmap




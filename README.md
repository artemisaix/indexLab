

# :microscope:Laboratorio de Indexación en MySQL :computer:
  
## 1. Introducción


**:dart: Objetivo del Laboratorio:**

Este laboratorio tiene como objetivo demostrar la importancia de la indexación en MySQL, mostrando cómo los índices pueden mejorar el rendimiento de las consultas, y qué aspectos debemos tener en cuenta para diseñarlos eficientemente.

**:memo: Requisitos Previos:**

* Tener instalado MySQL Server.
* Tener un cliente de MySQL (como MySQL Workbench o la línea de comandos).
* Haber creado la base de datos `world`.

**Estructura del Laboratorio:**

- Configuración inicial: creación de tablas y datos de prueba.
- Ejercicios prácticos centrados en la creación y optimización de índices.


--------------------
## 4. CONVERSION DE TIPOS :abc:  :arrows_counterclockwise: :1234:

Aquí tenemos un resumen de las reglas de conversión de tipos en MySQL:


:1234:  ==> :abc:  **Números a Cadenas:** Un número se convierte a su representación en cadena.

:abc:   ==>  :1234:   **Cadenas a Números:** Una cadena se convierte al número correspondiente.

:abc:  ==> :date:  :clock130:  **Cadenas a Fecha/Hora:** Una cadena válida se convierte a tipo de fecha u hora.

:no_entry_sign: ==> :zero:  **NULL:** Se convierte a 0 en contextos numéricos o a cadena vacía en contextos de cadena.

:1234: ==> :dollar:  **Enteros a Flotantes:** Se convierten automáticamente en contextos de punto flotante.


--------------------
  

## 2. Configuración Inicial :checkered_flag:

  

### 2.1 Crear la Base de Datos y las Tablas

  

```sql
-- Crear la base de datos
CREATE  DATABASE  IF  NOT  EXISTS world;
```
  
```sql
-- Utilizar la base de datos 'world'
USE world;
-- Creación de la tabla customers

DROP  TABLE  IF  EXISTS world.orders;
DROP  TABLE  IF  EXISTS world.customers;
CREATE  TABLE customers (
customer_id INT  NOT  NULL,
username VARCHAR(50)  NOT  NULL,
last_name VARCHAR(50)  NOT  NULL,
first_name VARCHAR(50)  NOT  NULL,
email VARCHAR(100)  NOT  NULL,
age INT  NOT  NULL,
city VARCHAR(50)  NOT  NULL,
active BOOLEAN NOT  NULL,
registration_date DATE  NOT  NULL,
comments TEXT,
amount DECIMAL(10,  2),
PRIMARY  KEY  (customer_id)
) ENGINE=InnoDB;


-- Creación de la tabla orders
DROP  TABLE  IF  EXISTS world.orders;
CREATE  TABLE orders (
order_id INT  NOT  NULL AUTO_INCREMENT,
customer_id INT  NOT  NULL,
customer_str CHAR(5),
order_date DATE  NOT  NULL,
amount DECIMAL(10,  2)  NOT  NULL,
PRIMARY  KEY  (order_id)
) ENGINE=InnoDB;
```

  

### 2.2 Insertar datos de prueba 'masivos'

#### - Primero necesitamos insertar unos registros iniciales que serviran como base para nuestro siguiente script de INSERT MASIVO

  

```sql
INSERT  INTO world.customers (customer_id ,username, last_name, first_name, email, age, city, active, registration_date, comments, amount)
VALUES
(1,'randomusr1',  'Smith',  'John',  'john.smith@example.com',  28,  'New York',  true,  '2023-01-15',  'Test comment 1',  100.00),
(2,'randomusr2',  'Doe',  'Jane',  'jane.doe@example.com',  32,  'Los Angeles',  false,  '2022-11-20',  'Test comment 2',  150.50),
(3,'randomusr3',  'Brown',  'Charlie',  'charlie.brown@example.com',  25,  'Chicago',  true,  '2023-07-10',  'Test comment 3',  200.75),
(4,'randomusr4',  'Johnson',  'Emily',  'emily.johnson@example.com',  45,  'Houston',  true,  '2020-03-12',  'Test comment 4',  250.00),
(5,'randomusr5',  'Johnson',  'Carla',  'emily.johnson@example.com',  45,  'Houston',  true,  '2020-03-12',  'Test comment 5',  250.00);

INSERT  INTO world.orders (customer_id, customer_str, order_date, amount)  VALUES
(1,  '1',  '2023-06-15',  100.00),
(1,  '1',  '2023-06-20',  150.50),
(2,  '2',  '2023-07-10',  200.75),
(3,  '3',  '2023-08-01',  250.00),
(4,  '4',  '2023-08-05',  300.00),
(5,  '5',  '2023-08-05',  350.00);
```

#### - Ahora, vamos a ejecutar los siguientes scripts AL MISMO TIEMPO unas 13 veces

```sql
-- Registros con email alta cardinalidad
INSERT  INTO world.customers (customer_id ,username, last_name, first_name, email, age, city, active, registration_date, comments, amount) SELECT customer_id +  (SELECT  MAX(customer_id)  FROM world.customers),
CONCAT('randomusr', customer_id +  (SELECT  MAX(customer_id)  FROM world.customers)),last_name, first_name,
CONCAT(CONCAT('randomusr', customer_id +  (SELECT  MAX(customer_id)  FROM world.customers)),  '@example.com'),
age, city, active, registration_date, comments, amount
FROM world.customers;

-- Ejecutar en conjunto con el insert de customers
INSERT  INTO world.orders (customer_id, customer_str, order_date, amount) SELECT
customer_id +  (SELECT  MAX(customer_id)  FROM world.orders),
customer_id +  (SELECT  MAX(customer_id)  FROM world.orders),
order_date,amount
FROM world.orders;
```

#### - PERFECTO!, ahora con esto tenemos lista nuestras tablas con un buen numero de registros en ellas.

```sql
SELECT  count(1)  FROM world.customers;
SELECT  count(1)  FROM world.orders;
```
 
#### - Vamos a crear los ìndices de ambas tablas

```sql
CREATE INDEX idx_email ON world.customers (email);
CREATE INDEX idx_username ON world.customers (username, city);
SHOW INDEX  FROM world.customers;
SHOW INDEX  FROM world.orders;
```

:speech_balloon: Como vemos la columna `customer_id` en `world.orders` no esta indexada, ùnicamente se usa como FK hacia la tabla padre `wold.customers`


--------------------
  

# 5. HANDS ON! :open_hands: :computer:

  

## ¡COMENCEMOS CON LOS EJERCICIOS!
### Ejercicio 1: LIKE Y WILDCARD PATTERN


:warning: **NOTA: MySQL puede elegir hacer full scan si la columna no tiene alta cardinalidad**

- Vamos a verificar si nuestro ìndice sobre email esta funcionando:
```sql
EXPLAIN  SELECT  *  
FROM world.customers 
WHERE email =  'emily.johnson@example.com';
```
Ahora vamos a hacer unas busquedas usando LIKE para ver si nuestro index sigue funcionando

```sql
EXPLAIN  SELECT  *  
FROM world.customers 
WHERE email LIKE  '%on@example.com';

EXPLAIN  SELECT  *  
FROM world.customers
WHERE email LIKE  'charlie%';
```

# QUIZ :interrobang:
:alarm_clock:
¿Podemos saber por que en una consulta si lo utiliza y en la otra no?

------
### Ejercicio 2: FUNCIONES SOBRE COLUMNAS INDEXADAS
```sql
EXPLAIN SELECT * 
FROM world.customers 
WHERE LOWER(email) = 'emily.johnson@example.com';
```
------
### Ejercicio 3: USO DE OR 
```sql
EXPLAIN SELECT * FROM world.customers c
WHERE username='randomusr5';
 
-- si funciona ¿por que?
EXPLAIN SELECT * FROM world.customers c
WHERE username='randomusr5' OR username='randomusr200';

-- no funciona ¿por que?
EXPLAIN SELECT * FROM world.customers c
WHERE username='randomusr5' OR city='Houston';
```

Cuando la consulta usa `OR` para combinar condiciones en columnas distintas (`username` y `city`), MySQL enfrenta un problema: no puede usar eficientemente el orden del índice para ambas columnas al mismo tiempo. En lugar de aprovechar el índice, MySQL puede optar por realizar un escaneo de tabla completa para cumplir con ambas condiciones.

------
### Ejercicio 4: DATATYPE MISSMATCH

Primero vamos a crear un ìndice sobre `customer_str`del tipo `VARCHAR`:

```sql
CREATE INDEX idx_customer_str ON world.orders (customer_str);
```
Ahora vamos a comprobar su eficiencia dependiendo el tipo de dato que estemos usando en la búsqueda:

```sql
-- DATATYPE MISSMATCH
EXPLAIN SELECT * FROM orders o
WHERE o.customer_str=10009;

EXPLAIN SELECT * FROM orders o
WHERE o.customer_str='10009';
```
Debemos tomar en cuenta el *Type Conversion in Expression Evaluation*
Lo podemos consultar más ampliamente en la documentaciòn oficial de MySQL:
https://dev.mysql.com/doc/refman/8.4/en/type-conversion.html

------
### Ejercicio 5: JOINS y uso de ìndices
Ahora vamos a probar la eficiencia de indexar las columnas que usamos durante nuestros `JOINS`en consultas más complejas que involucran mas de una tabla:

- Vamos a ejecutar la siguiente consulta con el `JOIN`usando una columna no indexada en `world.orders`:
```sql
EXPLAIN SELECT lt.first_name, lt.last_name, o.order_date, o.customer_str, o.amount
FROM world.customers lt
JOIN world.orders o ON lt.customer_id = o.customer_id
WHERE lt.email = 'emily.johnson@example.com';
```
Como podemos observar solo se hace uso eficiente del ìndice en la tabla `world.customers` por que estamos usando la PK, sin embargo, la columna customer_id, a pesar de ser del mismo tipo y tamaño, no esta indexada ni tiene una llave FK, por lo tanto hace un full scan, lo que no es optimo.

#### Vamos primero a crear un ìndice sobre `world.orders.customer_id`:
```sql
-- AQUI ESTAMOS CREANDO UN ÌNDICE SOBRE LA COLUMNA customer_id QUE SE USA EN EL JOIN 
CREATE INDEX idx_orders_customer_date ON world.orders (customer_id, order_date);
```
Ejecutemos nuevamente nuestra consulta y verifiquemos si mejora el plan de ejecuciòn:
```sql
EXPLAIN SELECT lt.first_name, lt.last_name, o.order_date, o.customer_str, o.amount
FROM world.customers lt
JOIN world.orders o ON lt.customer_id = o.customer_id
WHERE lt.email = 'emily.johnson@example.com';
```
Muy bien, ya mejoramos el uso del Join con ese ìndice. 

:warning: **NOTA:** Como buena práctica tambèn es bueno generar las FK necesarias en bases de datos Relacionales.

#### Ahora recordemos que creamos un ìndice sobre `world.customer.customer_str`y usaremos esta columna INDEXADA en el JOIN.
```sql
-- INDICE QUE CREAMOS PREVIAMENTE
CREATE INDEX idx_customer_str ON world.orders (customer_str);
```
# QUIZ :interrobang:

:alarm_clock: ¿Podemos saber si se harà uso efectivo de los ìndices en la consulta?

----------
```sql
EXPLAIN SELECT lt.first_name, lt.last_name, 
o.order_date, o.customer_str, o.amount
FROM world.customers lt
JOIN world.orders o ON lt.customer_id = o.customer_str
WHERE lt.email = 'emily.johnson@example.com';
```

------------------

## 6. INDICES COMPUESTOS

Ahora vamos a revisar propiedades un poco más complejas de los ìndices en **MySQL**, en esta sección estaremos viendo la mejor forma de utilizar y aprovechar nuestros **índices compuestos** en MySQL aprovechando la estructura **B-TREE**  :deciduous_tree: .


### Crear las as Tablas necesarias para el ejercicio

```sql
DROP TABLE IF EXISTS world.users;
CREATE TABLE world.users (
id INT NOT NULL AUTO_INCREMENT,
user_id INT NOT NULL,
last_name VARCHAR(50) NOT NULL,
first_name VARCHAR(50) NOT NULL,
email VARCHAR(100) NOT NULL,
age INT NOT NULL,
city VARCHAR(50) NOT NULL,
active BOOLEAN NOT NULL,
registration_date DATE NOT NULL,
PRIMARY KEY (id));
```
### Creamos el ìndice compuesto
```sql
CREATE INDEX idx_user_city_last 
ON world.users (user_id, city, last_name);
```
### Insertamos datos de Ejemplo
```sql
INSERT INTO world.users
(user_id, last_name, first_name, email, age, city, active, registration_date)
VALUES
(1001, 'Smith', 'John', 'john.smith@example.com', 28, 'New York', true, '2023-01-15'),
(1002, 'Doe', 'Jane', 'jane.doe@example.com', 32, 'Los Angeles', false, '2022-11-20'),
(1003, 'Brown', 'Charlie', 'charlie.brown@example.com', 25, 'Chicago', true, '2021-08-05'),
(1004, 'Johnson', 'Emily', 'emily.johnson@example.com', 45, 'Houston', true, '2020-03-12'),
(1005, 'Williams', 'Michael', 'michael.williams@example.com', 37, 'Phoenix', false, '2019-12-19'),
(1006, 'Jones', 'Sarah', 'sarah.jones@example.com', 29, 'San Antonio', true, '2023-04-23'),
(1007, 'Garcia', 'David', 'david.garcia@example.com', 34, 'Dallas', false, '2021-05-10'),
(1008, 'Martinez', 'Laura', 'laura.martinez@example.com', 30, 'San Diego', true, '2022-02-14'),
(1009, 'Hernandez', 'Carlos', 'carlos.hernandez@example.com', 40, 'San Jose', false, '2018-09-25'),
(1010, 'Lopez', 'Sophia', 'sophia.lopez@example.com', 27, 'Austin', true, '2020-07-17');
```
Una vez insertados los registros iniciales, podemos aumentar la cantidad de registros ejecutando este script:

```sql
INSERT INTO world.users (user_id, last_name, first_name, 
email, age, city, active, registration_date)
SELECT 
    user_id + (SELECT MAX(user_id) FROM world.users),
    last_name, 
    first_name, 
    CONCAT(first_name, '.', last_name, '@example.com'), 
    age, 
    city, 
    active, 
    registration_date
FROM world.users;
```
### Ejercicio 6: :white_check_mark: Orden en el INDEX compuesto

En este ejercicio vamos a ver como un mismo índice nos sirve para cubrir diferentes tipos de consultas sin necesidad de crear índices con las diferentes combinaciones de columnas, recordemos, es importante el ORDEN en el que elegimos diseñar el índice  MAS SELECTIVO --> MENOS SELECTIVO, y lo mismo aplica para cuando estemos generando nuestras Consultas:

```sql
-- Uso del Primer Campo del Índice
EXPLAIN SELECT user_id, first_name,last_name,
email,age, active
FROM world.users
WHERE user_id = 1006;

-- Uso del Primer y Segundo Campo del Índice
EXPLAIN SELECT user_id, first_name,last_name,
email,age, active
FROM world.users
WHERE user_id = 1002
AND city = 'Los Angeles';

-- Uso de Todos los Campos del Índice
EXPLAIN SELECT user_id, first_name,last_name,
email,age, active
FROM world.users
WHERE user_id = 1001
AND city = 'New York'
AND last_name = 'Smith';

-- Rango en el Segundo Campo del Índice
EXPLAIN SELECT user_id, first_name,last_name,
email,age, active
FROM world.users
WHERE user_id = 1004
AND city = 'Houston'
AND last_name >= 'J' AND last_name < 'K';

-- Uso de BETWEEN
EXPLAIN SELECT user_id, first_name,last_name,
email,age, active
FROM world.users
WHERE user_id BETWEEN 1002 AND 1004
AND city = 'New York';

-- Uso de BETWEEN, LIKE Y RANGOS
EXPLAIN SELECT user_id, first_name,last_name,
email,age, active FROM world.users
WHERE user_id BETWEEN 1002 AND 1004
AND city like  'New%'
AND last_name >= 'J' AND last_name < 'K';
```

### Ejercicio 7: :white_check_mark: BUEN USO de las propiedades de B-TREE  :deciduous_tree:

 Ahora que vimos lo básico sobre como consultar tablas que tengan índices compuestos, vamos a adentrarnos un poco más en el funcionamiento de la estructura *B-TREE*:deciduous_tree:  con los siguientes ejemplos:
 
```sql
-- B TREE
-- Uso del Primer Campo del Índice con LIKE
EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
AND last_name LIKE 'Sm%';

-- Condiciones en Prefijo Izquierdo del Índice y 
-- columna no indexada age
EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
AND city = 'New York' 
AND age = 28;

EXPLAIN SELECT * 
FROM world.users 
WHERE age = 30 
AND user_id = 1002;

EXPLAIN SELECT * 
FROM world.users 
WHERE city = 'New York' 
AND user_id = 1002;

EXPLAIN SELECT * 
FROM world.users 
WHERE last_name = 'Smith' 
AND city = 'New York' 
AND user_id = 1002;

EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1002 
and last_name = 'Smith';
```
----------------
Hasta aquí vemos algunas posiblidades del indice compuesto y la estructura B-TREE... ahora veamos ejemplos más interesantes:
### Ejercicio 8: :white_check_mark: OR en búsqueda de campos indexados
```sql
-- Uso del Índice en una Parte y No en Otra: Usa el índice en 
-- (user_id, city) para la primera parte 
-- y (user_id, last_name) para la segunda

EXPLAIN SELECT * 
FROM world.users 
WHERE (user_id = 1001 AND city = 'New York') 
OR 
(user_id = 1002 AND last_name = 'Doe');

EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
OR 
(user_id = 1002 AND city = 'Los Angeles');

```
![Spicy memes - Imgflip](https://i.imgflip.com/58kv19.jpg)


Entiendo que habrá algunas dudas, comprensible, aquí la explicación:

1.  **`SELECT * FROM world.users WHERE (user_id = 1001 AND city = 'New York') OR (user_id = 1002 AND last_name = 'Doe');`**
    
    -   Esta consulta  utiliza `OR`, pero cada condición dentro de los paréntesis se aplica a un prefijo de las columnas del índice en el orden correcto (`user_id` y `city` en la primera condición,  `user_id` y `last_name` en la segunda).
    -   MySQL puede utilizar el índice para buscar eficientemente filas que cumplan cualquiera de las dos condiciones compuestas.
2.  **`SELECT * FROM world.users WHERE user_id = 1001 OR (user_id = 1002 AND city = 'Los Angeles');`**
    
    -   Esta consulta es similar a la anterior. La primera condición (`user_id = 1001`) utiliza solo la primera columna del índice, y la segunda condición compuesta utiliza un prefijo de las columnas del índice en el orden correcto (`user_id` y `city`).
    -   MySQL puede utilizar el índice para optimizar ambas partes de la consulta.
### Ejercicio 9: :white_check_mark: OR combinando columnas indexadas y columnas sin INDEX
```sql
-- Condición con OR y AND en el Índice: Usa el índice 
-- en (user_id) pero con una combinación OR y AND
EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
OR 
(age = 30 AND user_id = 1002);
```

![Confused Nick Young... - Meme templates HD | Facebook](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRFxrjLlgYqejPKaNsY95739Oqmm33dXWd_tg&s)

----------------
**Análisis de las consultas OR con columna no INDEXADA incluida:**

MySQL sí utiliza el índice correctamente en esta consulta, a pesar de que la columna `age` no forma parte del índice y aparece primero dentro de los paréntesis. **¿Por qué?**

La clave está en entender cómo MySQL procesa las condiciones lógicas y cómo optimiza las consultas. Aquí hay algunos puntos clave:

1.  **Optimización de consultas:** El optimizador de consultas de MySQL analiza la consulta completa y decide la mejor manera de ejecutarla, considerando factores como los índices disponibles, las estadísticas de la tabla y las condiciones de filtrado.
    
2.  **Reordenamiento de condiciones:** El optimizador de consultas tiene la libertad de reordenar las condiciones dentro de una consulta para mejorar el rendimiento, siempre y cuando el reordenamiento no afecte el resultado final de la consulta.
    
3.  **Precedencia de operadores:** En la expresión `(age = 30 AND user_id = 1002)`, el operador `AND` tiene mayor precedencia que el operador `OR`. Esto significa que MySQL evaluará primero la condición compuesta entre paréntesis antes de combinarla con la condición `user_id = 1001` usando `OR`.
    
4.  **Uso del índice:**
    
    -   La condición `user_id = 1001` puede ser evaluada directamente utilizando el índice `idx_user_city_last`, ya que `user_id` es la primera columna del índice.
    -   Dentro de los paréntesis, aunque `age` no está indexada, la condición `user_id = 1002` también puede utilizar el mismo índice.
    -   El optimizador de consultas puede reordenar las condiciones dentro de los paréntesis para evaluar primero `user_id = 1002` utilizando el índice, y luego filtrar further las filas resultantes por `age = 30`.

### Continuemos ....

### :x: MAL USO de las propiedades de B-TREE  :deciduous_tree: 

Ahora, ya que vimos lo que sí debemos hacer, no esta de más conocer algunas prácticas que nos podrían estar jugando en contra para aprovechar los índices de nuestras tablas:

***PERO ANTES!!***

# QUIZ :interrobang:

:alarm_clock: ¿A que nos referimos con el índice más la izquierda de un índice compuesto?

----------

### Ejercicio 10: :x: No hacer uso del prefijo mas a la izquierda 

```sql
-- Uso del Segundo Campo del Índice Solo
EXPLAIN SELECT * 
FROM world.users 
WHERE city = 'New York';

-- Índice No Usado en Prefijo Izquierdo:No usa el índice 
-- (user_id, city, last_name) porque no incluye user_id
EXPLAIN SELECT * 
FROM world.users 
WHERE city = 'Houston' 
AND last_name = 'Smith';
```
### Ejercicio 11: :x: Mal uso del OR
```sql
-- Índice No Usado en Ambas Partes de la Condición: 
-- No usa el índice (user_id, city, last_name) porque 
-- el OR no puede ser optimizado con el índice
EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
OR age = 30;

EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
OR city = 'Los Angeles';
```
### Ejercicio 12: :x: Mal uso de los RANGOS
```sql
-- Rangos mal definidos, no acotados
EXPLAIN SELECT * FROM world.users WHERE user_id BETWEEN 1002 AND 1100 AND city = 'New York'; -- No funciona :(

EXPLAIN SELECT * FROM world.users WHERE user_id BETWEEN 1002 AND 1004 AND city = 'New York'; -- Si funciona :) ¿?
```
# QUIZ :interrobang:

:alarm_clock: ¿Por qué el ìndice nos funciona en el segundo escenario pero no en primero?

----------

# 6. CASOS DE ESTUDIO
A continuación tenemos varios casos de estudio para analizar y encontrar alguna manera de poder mejorar el performance de las consultas y el uso eficiente de los recursos en general: 

## CASO 1

Nos solicitan optimizar un flujo en el cual tenemos demasiado consumo de CPU para operaciones de actualización e inserción de datos.

#### - Primero necesitamos preparar nuestro escenario para el caso de estudio

```sql

DROP TABLE IF EXISTS world.users;
CREATE TABLE world.users (
    id INT NOT NULL AUTO_INCREMENT,
    user_id INT NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    age INT NOT NULL,
    city VARCHAR(50) NOT NULL,
    active BOOLEAN NOT NULL,
    registration_date DATE NOT NULL,
    PRIMARY KEY (id)
);
```
#### Ahora vamos a crear los índices
```sql
CREATE INDEX idx_user_city ON world.users (user_id, city);
CREATE INDEX idx_user_city_active ON world.users (user_id, city, active);
```
#### Vamos a insertar algunos datos de prueba
```sql
INSERT INTO world.users
(user_id, last_name, first_name, email, age, city, active, registration_date)
VALUES
(1001, 'Smith', 'John', 'john.smith@example.com', 28, 'New York', true, '2023-01-15'),
(1002, 'Doe', 'Jane', 'jane.doe@example.com', 32, 'Los Angeles', false, '2022-11-20'),
(1003, 'Brown', 'Charlie', 'charlie.brown@example.com', 25, 'Chicago', true, '2021-08-05'),
(1004, 'Johnson', 'Emily', 'emily.johnson@example.com', 45, 'Houston', true, '2020-03-12'),
(1005, 'Williams', 'Michael', 'michael.williams@example.com', 37, 'Phoenix', false, '2019-12-19'),
(1006, 'Jones', 'Sarah', 'sarah.jones@example.com', 29, 'San Antonio', true, '2023-04-23'),
(1007, 'Garcia', 'David', 'david.garcia@example.com', 34, 'Dallas', false, '2021-05-10'),
(1008, 'Martinez', 'Laura', 'laura.martinez@example.com', 30, 'San Diego', true, '2022-02-14'),
(1009, 'Hernandez', 'Carlos', 'carlos.hernandez@example.com', 40, 'San Jose', false, '2018-09-25'),
(1010, 'Lopez', 'Sophia', 'sophia.lopez@example.com', 27, 'Austin', true, '2020-07-17');
```
#### Una vez insertados los registros iniciales, podemos aumentar la cantidad de registros ejecutando este script:

```sql
-- Ejecutar unas 10-13 veces
INSERT INTO world.users (user_id, last_name, first_name, 
email, age, city, active, registration_date)
SELECT 
    user_id + (SELECT MAX(user_id) FROM world.users),
    last_name, 
    first_name, 
    CONCAT(first_name, '.', last_name, '@example.com'), 
    age, 
    city, 
    active, 
    registration_date
FROM world.users;
```

Vale, ahora vamos a iniciar a analizar este escenario. Lo primero que tenemos que hacer es identificar a que caso nos estamos enfrentando.

 **- Podemos consultar los índices actuales de la tabla:**
```sql
SHOW INDEX FROM world.users;
```
Podemos ver que hay columnas que se repiten en ambos índices, esto puede _sugerir_ que estemos ante un caso de índices redundantes. :eyes: **OJO**, esto no significa que tengamos precisamente una _sobreindexación_ en la tabla.

:speech_balloon: _Si tenemos permisos para consultar tablas de administración podemos consultar las estadísticas de los índices_

 **- Ahora podemos ejecutar un par de consultas para ver que tal se comportan los ìndices**

```sql
/*
 El optimizador de MySQL puede tener que elegir entre dos índices muy similares, lo que puede llevar a decisiones subóptimas.
 */
EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
AND city = 'New York';

EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
AND city = 'New York' 
AND active = true;
```
Como vemos, el optimizador de consultas se decide a utilizar únicamente uno de los ìndices de la tabla, esto quiere decir, que hay un ìndice ***de màs*** que no esta siendo aprovechado y solo nos esta generando procesamiento adicional con el mantenimiento de estadísticas y baja en el performance de operaciones de Escritura como INSERTS, DELETE y UPDATES, así como espacio adicional en nuestro storage.

# QUIZ :interrobang:

:alarm_clock: ¿A qué caso nos enfrentamos?

----------

<label> <input type="radio" name="opcion" value="1. Índices insuficientes">1.  Índices insuficientes </label><br>

<label> <input type="radio" name="opcion" value="2. Índices redundantes" checked> 2. Índices redundantes </label><br>

<label> <input type="radio" name="opcion" value="3 Sobreindexación"> 3. Sobreindexación </label><br>

----------

Una vez que sabemos a que nos enfrentamos, tenemos que realizar diversos análisis y pruebas aprovechando el conocimiento que adquirimos en los ejercicios anteriores para tratar de encontrar alguna mejora en el performance de la tabla aplicando las mejoras prácticas en cuanto al diseño de índices.

# QUIZ :interrobang:

:alarm_clock: ¿Qué podríamos hacer para mejorar los índices en esta tabla? Elige la opción que te parezca más factible:


----------
**Opciones:**

1.  **Crear un nuevo índice que incluya todas las columnas de la tabla.**
2.  **Mantener ambos índices tal como están y modificar las consultas en el aplicativo**
3.  **Eliminar el índice `idx_user_city_active` ya que `idx_user_city` lo cubre completamente.**
![Mathematical Thoughts in the Hangover | Math in Movies](https://i.ytimg.com/vi/Mb1x_tRg2mc/hq720.jpg?sqp=-oaymwEhCK4FEIIDSFryq4qpAxMIARUAAAAAGAElAADIQj0AgKJD&rs=AOn4CLAgAq2Ne4IA5MMNbCupAR-m-g1Yqw)

 **- Listo, hora de ejecutar nuestro plan de acción y realizar las pruebas necesarias para validar su efectividad**
```sql
-- POSIBLE SOLUCIÒN
DROP INDEX idx_user_city ON users;

EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
AND city = 'New York';

EXPLAIN SELECT * 
FROM world.users 
WHERE user_id = 1001 
AND city = 'New York' 
AND active = true;
```

:speech_balloon: _No siempre nuestra primera solución es la indicada, recuerda tener paciencia y recuerda que hasta los mejores han triunfado mediante prueba y error :relaxed:_

## CASO 2 

Nos solicitan revisar un crecimiento excesivo en el storage del servidor de la base de datos, así como lentitud de algunos JOBS que insertan nuevos datos y actualizan datos de la siguiente tabla,

 **- Creamos la tabla para nuestro segundo caso de estudio:**
 ```sql
DROP TABLE IF EXISTS world.clientes;
CREATE TABLE world.clientes(
    id INT NOT NULL AUTO_INCREMENT,
    user_id INT NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    age INT NOT NULL,
    city VARCHAR(50) NOT NULL,
    active BOOLEAN NOT NULL,
    registration_date DATE NOT NULL,
    PRIMARY KEY (id)
);
 ```
  **- Creamos los indices para nuestra simulación**
  ```sql
CREATE INDEX idx_redundante1 ON world.clientes (user_id,email,active);
CREATE INDEX idx_redundante2 ON world.clientes (user_id,email,last_name);
CREATE INDEX idx_redundante3 ON world.clientes (user_id,age,last_name);
CREATE INDEX idx_redundante4 ON world.clientes (email,user_id,age);
CREATE INDEX idx_redundante5 ON world.clientes (email,age);
  ```

** -Insertamos datos para nuestra tabla**
```sql
INSERT INTO world.clientes (user_id, last_name, first_name, email, age, city, active, registration_date) VALUES
(1001, 'Smith', 'John', 'john.smith@example.com', 28, 'New York', true, '2023-01-15'),
(1002, 'Doe', 'Jane', 'jane.doe@example.com', 32, 'Los Angeles', false, '2022-11-20'),
(1003, 'Brown', 'Charlie', 'charlie.brown@example.com', 25, 'Chicago', true, '2021-08-05'),
(1004, 'Johnson', 'Emily', 'emily.johnson@example.com', 45, 'Houston', true, '2020-03-12'),
(1005, 'Williams', 'Michael', 'michael.williams@example.com', 37, 'Phoenix', false, '2019-12-19'),
(1006, 'Smith', 'Sarah', 'sarah.jones@example.com', 29, 'San Antonio', true, '2023-04-23'),
(1007, 'Smith', 'David', 'david.garcia@example.com', 34, 'Dallas', false, '2021-05-10'),
(1008, 'Martinez', 'Laura', 'laura.martinez@example.com', 30, 'San Diego', true, '2022-02-14'),
(1009, 'Hernandez', 'Carlos', 'carlos.hernandez@example.com', 40, 'San Jose', false, '2018-09-25'),
(1010, 'Lopez', 'Sophia', 'sophia.lopez@example.com', 27, 'Austin', true, '2020-07-17');

-- Ejecutar unas 10-13 veces
INSERT INTO world.clientes (user_id, last_name, first_name, email, age, city, active, registration_date)
SELECT 
    user_id + (SELECT MAX(user_id) FROM world.clientes),
    last_name, 
    first_name, 
    CONCAT(first_name, '.', last_name, '@example.com'), 
    age, 
    city, 
    active, 
    registration_date
FROM world.clientes;
```
** -Validamos como se estan usando los índices actuales con diferentes consultas y escenarios**

```sql
EXPLAIN SELECT * 
FROM world.clientes
WHERE user_id = 1001 
AND email = 'john.smith@example.com' 
AND active = true;

EXPLAIN SELECT * 
FROM world.clientes
WHERE user_id = 1001 
AND email = 'john.smith@example.com' 
AND last_name = 'Smith';

EXPLAIN SELECT * 
FROM world.clientes
WHERE user_id = 1001 
AND age=28 
AND last_name = 'Smith';

-- Proposito distinto
EXPLAIN SELECT * 
FROM world.clientes
WHERE email = 'john.smith@example.com' 
AND age=28 ;

-- Excepciones a la Regla
EXPLAIN SELECT * 
FROM world.clientes
WHERE email = 'john.smith@example.com' 
AND user_id = 1001 
AND age=28 ;
```

** -Ahora que vimos que hay muchos índices que se estan ignorando podemos plantear una primera solución:**

```sql
-- Índices optimizados
DROP INDEX idx_redundante1 ON world.clientes;
DROP INDEX idx_redundante2 ON world.clientes;
DROP INDEX idx_redundante3 ON world.clientes;
DROP INDEX idx_redundante4 ON world.clientes;

CREATE INDEX idx_user_email_active_lastname 
ON world.clientes (user_id, email, active, last_name);
```

Verificamos nuestros nuevos índices

```sql
show index from world.clientes;
```

** -Ejecutamos nuevamente nuestras consultas y vemos el nuevo comportamiento**
```sql
EXPLAIN SELECT * 
FROM world.clientes
WHERE user_id = 1001 
AND email = 'john.smith@example.com' 
AND active = true;

EXPLAIN SELECT * 
FROM world.clientes
WHERE user_id = 1001 
AND email = 'john.smith@example.com' 
AND last_name = 'Smith';

EXPLAIN SELECT * 
FROM world.clientes
WHERE user_id = 1001 
AND age=28 
AND last_name = 'Smith';

-- Proposito distinto
EXPLAIN SELECT * 
FROM world.clientes
WHERE email = 'john.smith@example.com' 
AND age=28 ;

-- Excepciones a la Regla
EXPLAIN SELECT * 
FROM world.clientes
WHERE email = 'john.smith@example.com' 
AND user_id = 1001 
AND age=28 ;
```
:speech_balloon: Si cuentas con permisos de administración, una buena práctica es programar una actualización de los índices después de hacer cambios en las estructuras de las tablas.
```sql
ANALYZE TABLE nombre_de_tu_tabla;
```

**Si estamos satisfechos con los resultados se puede dejar hasta aquí la optimización**

# :tada:¡FELICIDADES!:confetti_ball:
### Llegamos al final de este laboratorio. Muchas gracias por tu atención, espero que de todo lo que pudimos practicar te puedas llevar algo para aplicar en tus proyectos.

## :page_facing_up: DOCUMENTACIÓN OFICIAL
### Optimization and Indexes
https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html
### Type Conversion in Expression Evaluation
https://dev.mysql.com/doc/refman/8.4/en/type-conversion.html
### Comparison of B-Tree and Hash Indexes
https://dev.mysql.com/doc/refman/8.4/en/index-btree-hash.html


# Mini Project Gambling SQL

```sql
-- Setting the working database
USE gambling;
USE student_school;

-- Pregunta 01: Usando la tabla o pestaña de clientes, 
-- por favor escribe una consulta SQL que muestre Título, Nombre y Apellido y Fecha de Nacimiento para cada uno de los clientes. 
-- No necesitarás hacer nada en Excel para esta.
SELECT Title, FirstName, LastName, DateOfBirth FROM customer;

-- Pregunta 02: Usando la tabla o pestaña de clientes, 
-- por favor escribe una consulta SQL que muestre el número de clientes en cada grupo de clientes (Bronce, Plata y Oro). 
-- Puedo ver visualmente que hay 4 Bronce, 3 Plata y 3 Oro pero si hubiera un millón de clientes ¿cómo lo haría en Excel?
SELECT CustomerGroup, COUNT(*) AS number_customer 
FROM customer
GROUP BY CustomerGroup;

-- Pregunta 03: El gerente de CRM me ha pedido que proporcione una lista completa de todos los datos para esos 
-- clientes en la tabla de clientes pero necesito añadir el código de moneda de cada jugador para que pueda 
-- enviar la oferta correcta en la moneda correcta. 
-- Nota que el código de moneda no existe en la tabla de clientes sino en la tabla de cuentas. 
-- Por favor, escribe el SQL que facilitaría esto. 
-- ¿Cómo lo haría en Excel si tuviera un conjunto de datos mucho más grande?
SELECT c.*, a.CurrencyCode
FROM customer AS c
LEFT JOIN account AS a
ON c.CustId = a.CustId;

-- Pregunta 04: Ahora necesito proporcionar a un gerente de producto un informe resumen que muestre, por producto y por día, 
-- cuánto dinero se ha apostado en un producto particular. 
-- TEN EN CUENTA que las transacciones están almacenadas en la tabla de apuestas y hay un código de producto en esa tabla 
-- que se requiere buscar (classid & categoryid) para determinar a qué familia de productos pertenece esto. 
-- Por favor, escribe el SQL que proporcionaría el informe. 
-- Si imaginas que esto fue un conjunto de datos mucho más grande en Excel, ¿cómo proporcionarías este informe en Excel?
SELECT 
    p.product, 
    STR_TO_DATE(b.BetDate, '%d/%m/%y') AS bet_date_format, 
    SUM(b.Bet_Amt) AS total_bet
FROM betting AS b
JOIN product AS p
ON b.ClassID = p.CLASSID AND b.CategoryId = p.CATEGORYID 
GROUP BY p.product, bet_date_format
ORDER BY p.product, bet_date_format;

-- Pregunta 05: Acabas de proporcionar el informe de la pregunta 4 al gerente de producto, 
-- ahora él me ha enviado un correo electrónico y quiere que se cambie. 
-- ¿Puedes por favor modificar el informe resumen para que solo resuma las transacciones que ocurrieron el 1 de noviembre o después
-- y solo quiere ver transacciones de Sportsbook. 
-- Nuevamente, por favor escribe el SQL abajo que hará esto. 
-- Si yo estuviera entregando esto vía Excel, ¿cómo lo haría?
SELECT 
    p.product, 
    STR_TO_DATE(b.BetDate, '%d/%m/%y') AS bet_date_format, 
    SUM(b.Bet_Amt) AS total_bet
FROM betting AS b
JOIN product AS p
ON b.ClassID = p.CLASSID AND b.CategoryId = p.CATEGORYID 
WHERE b.BetDate >= '2020-11-01' AND p.product = 'Sportsbook'
GROUP BY p.product, bet_date_format
ORDER BY p.product, bet_date_format;

-- Pregunta 06: Como suele suceder, el gerente de producto ha mostrado su nuevo informe a su director 
-- y ahora él también quiere una versión diferente de este informe. 
-- Esta vez, quiere todos los productos pero divididos por el código de moneda y el grupo de clientes del cliente, 
-- en lugar de por día y producto. 
-- También le gustaría solo transacciones que ocurrieron después del 1 de diciembre. 
-- Por favor, escribe el código SQL que hará esto.

CREATE OR REPLACE VIEW acc_bet AS
SELECT a.CustID, a.CurrencyCode, b.Bet_Amt 
FROM betting AS b
JOIN account AS a
ON b.AccountNo = a.AccountNo
WHERE b.BetDate >= '2012-12-01';

SELECT * FROM acc_bet;

CREATE OR REPLACE VIEW acc_bet_cus AS
SELECT a.CurrencyCode, c.CustomerGroup, SUM(a.Bet_Amt) AS TotalAmount 
FROM acc_bet AS a
JOIN customer AS c
ON a.CustID = c.CustID
GROUP BY a.CurrencyCode, c.CustomerGroup
ORDER BY totalAmount DESC;

SELECT * FROM acc_bet_cus;

-- Pregunta 07: Nuestro equipo VIP ha pedido ver un informe de todos los jugadores independientemente de si han hecho algo en el marco de tiempo completo o no. 
-- En nuestro ejemplo, es posible que no todos los jugadores hayan estado activos. 
-- Por favor, escribe una consulta SQL que muestre a todos los jugadores Título, Nombre y Apellido y un resumen de su cantidad de apuesta para el período completo de noviembre.

CREATE OR REPLACE VIEW cust_acc AS
SELECT cu.Title, cu.FirstName, cu.LastName, ac.AccountNo
FROM customer AS cu
JOIN account AS ac
ON cu.CustId = ac.CustId;

SELECT * FROM cust_acc;

CREATE OR REPLACE VIEW bet_acc AS
SELECT cu.Title, cu.FirstName, cu.LastName, be.Bet_Amt, STR_TO_DATE(be.BetDate, '%d/%m/%y') AS format_date
FROM cust_acc AS cu
JOIN betting AS be
ON cu.AccountNo = be.AccountNo
WHERE MONTH(format_date) = 11;

SELECT * FROM bet_acc;

-- Pregunta 08: Nuestros equipos de marketing y CRM quieren medir el número de jugadores que juegan más de un producto. 
-- ¿Puedes por favor escribir 2 consultas, una que muestre el número de productos por jugador 
-- y otra que muestre jugadores que juegan tanto en Sportsbook como en Vegas?

SELECT c.FirstName, COUNT(p.product) AS products_played
FROM customer AS c
INNER JOIN account AS a ON c.CustId = a.CustId
INNER JOIN betting AS b ON a.AccountNo = b.AccountNo
INNER JOIN product AS p ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
GROUP BY c.FirstName;

SELECT c.FirstName
FROM customer AS c
INNER JOIN account AS a ON c.CustId = a.CustId
INNER JOIN betting AS b ON a.AccountNo = b.AccountNo
INNER JOIN product AS p ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
WHERE p.product IN ('Sportsbook', 'Vegas')
GROUP BY c.FirstName
HAVING COUNT(DISTINCT p.product) = 2;

-- Pregunta 09: Ahora nuestro equipo de CRM quiere ver a los jugadores que solo juegan un producto, 
-- por favor escribe código SQL que muestre a los jugadores que solo juegan en sportsbook, usa bet_amt > 0 como la clave. 
-- Muestra cada jugador y la suma de sus apuestas para ambos productos.

SELECT c.FirstName, p.product
FROM customer AS c
INNER JOIN account AS a ON c.CustId = a.CustId
INNER JOIN betting AS b ON a.AccountNo = b.AccountNo
INNER JOIN product AS p ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
WHERE p.product = 'Sportsbook' 
GROUP BY c.FirstName
HAVING COUNT(DISTINCT p.product) = 1;

-- Pregunta 10: La última pregunta requiere que calculemos y determinemos el producto favorito de un jugador. 
-- Esto se puede determinar por la mayor cantidad de dinero apostado. 
-- Por favor, escribe una consulta que muestre el producto favorito de cada jugador

SELECT
    c.FirstName, 
    p.product, 
    RANK() OVER (PARTITION BY c.FirstName ORDER BY COUNT(p.product) DESC) AS 'Rank'
FROM customer AS c
INNER JOIN account AS a ON c.CustId = a.CustId
INNER JOIN betting AS b ON a.AccountNo = b.AccountNo
INNER JOIN product AS p ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
GROUP BY c.FirstName
ORDER BY c.FirstName, 'Rank';

-- Pregunta 11: Escribe una consulta que devuelva a los 5 mejores estudiantes basándose en el GPA
SELECT *, DENSE_RANK() OVER(ORDER BY GPA DESC) AS 'Rank_GPA'
FROM student
LIMIT 5;

-- Pregunta 12: Escribe una consulta que devuelva el número de estudiantes en cada escuela. (¡una escuela debería estar en la salida incluso si no tiene estudiantes!)
SELECT sc.school_name, COUNT(student_id) AS number_students
FROM student AS st
INNER JOIN school AS sc
ON st.school_id = sc.school_id
GROUP BY sc.school_name;

-- Pregunta 13: Escribe una consulta que devuelva los nombres de los 3 estudiantes con el GPA más alto de cada universidad.
SELECT st.student_name, sc.school_name, DENSE_RANK() OVER(ORDER BY GPA DESC) AS 'Rank_GPA'
FROM student AS st
INNER JOIN school as sc
ON st.school_id = sc.school_id
WHERE 'Rank_GPA' <= 3;
```



Most of these notes are taken from online tutorial [MySQL Tutorial for Beginners (3h10m) - March 2019](https://www.youtube.com/watch?v=7S_tz1z_5bA) by **Programming with Mosh**.

# Contents:
* [SELECT](#SELECT)
  * [IN, BETWEEN](#SELECT_IN)
  * [LIKE](#SELECT_LIKE)
  * [REGEXP](#SELECT_REGEXP)
  * [IS NULL](#SELECT_IS_NULL)
  * [ORDER BY](#SELECT_ORDERBY)
  * [LIMIT](#SELECT_LIMIT)
* [INNER JOIN](#INNERJOIN)
  * [JOIN Multiple Tables](#INNERJOIN_MULTIPLE)
  * [COMPOUND JOIN](#INNERJOIN_COMPOUND)
* [OUTTER JOIN](#OUTTERJOIN)
  * [USING](#USING)
* [UNIONS](#UNIONS)
* [INSERT INTO](#INSERT)
  * [INSERT MULTIPLE VALUES INTO A TABLE](#INSERT_MULTIPLE)
  * [INSERT DATAINTO MULTIPLE TABLES](#INSERT_MULTIPLE2)


## <a name="SELECT"></a>SELECT
```
SELECT * FROM Customers WHERE Name = 'Andrew';
```

```
SELECT
  last_name,
  first_name,
  points,
  points*10+100 AS discount_factor
FROM Customers;
```

If we have duplicates in a column (eg city: New York for more than 2 people) and we don't want to display them, we use DISTINCT:
```
SELECT DISTINCT City
FROM People;
```

**Dates** are written within quotes: 'year-month-day'
```
SELECT *
FROM Customers
WHERE (birth_date > '1990-01-01' AND points > 1000) OR state = 'LA';
```

### <a name="SELECT_IN"></a>SELECT: IN, BETWEEN
```
SELECT *
FROM Customers
WHERE state IN ('VA','GA','LA');

/* is the same as:
WHERE state = 'VA' OR STATE = 'GA' OR STATE = 'LA;
*/
```
```
SELECT *
FROM Customers
WHERE points BETWEEN 1000 AND 3000;

/* is the same as (while includings the ends of range):
WHERE points >= 1000 OR points <= 3000;
*/
```

### <a name="SELECT_LIKE"></a>SELECT: LIKE
Eg.: For a person whose last name starts with 'b'/'B' (it doesn't matter if it's lower or upper case):
```
SELECT * FROM Customers WHERE last_name LIKE 'b%';
```
Eg.: Person whose name ends with '%eanu'
Eg.: Person whose name contains letter '%z%' (it doesn't matter if letter 'z' is at first, middle or end)
Eg.: Person whose name is composed from 3 letters and the second letter is 'n': '\_n\_' (eg Ana)
```
SELECT Prenume, Nume FROM Pacienti WHERE Prenume LIKE '_n_';
```

### <a name="SELECT_REGEXP"></a>SELECT: REGEXP
```
SELECT * FROM Pacienti
WHERE Nume REGEXP 'escu';

/* is the same as WHERE Nume LIKE '%escu%';*/
```
Alte exemple:
* REGEXP '^Munte' --- string-ul trebuie sa inceapa cu 'Munte..."
* REGEXP 'escu$' --- string-ul se termina cu '...escu'
* REGEXP 'eanu|escu' --- string care contine 'eanu' sau 'escu'
* REGEXP '[gim]e' --- orice persoana care contine 'ge' OR 'ie' OR 'me' (echivalent cu REGEXP 'ge|ie|me')
* REGEXP '[a-h]e' --- 'a' to 'h', deci fiecare combinatie 'ae' 'be' ... 'he'

### <a name="SELECT_IS_NULL"></a>SELECT: IS NULL
Get all the records with **missing values**:
```
SELECT * FROM Pacienti
WHERE IdSectie IS NULL;
```

### <a name="SELECT_ORDERBY"></a>SELECT: ORDER BY
By default, liniile sunt ordonate dupa id (Dacă selectezi meniul ALTER TABLE din MySQL al unei tabele, IdPacient va avea langa o cheie aurie - anume PRIMARY KEY)
```
SELECT * FROM Pacienti ORDER BY Nume;
```
Sorteaza descrescator dupa judet apoi dupa nume crescator:
```
SELECT * FROM Pacienti
ORDER BY Judet DESC Nume;
```
```
SELECT first_name, last_name
FROM Customers
ORDER BY 1,2; --- sorteaza pirmele doua coloane precizate in ordine imediat dupa SELECT (de evitat)
```
```
SELECT *, quantity*unit_price AS 'total price'
FROM Order_items
WHERE order_id=2
ORDER BY 'total price' DESC;
```

### <a name="SELECT_LIMIT"></a>SELECT: LIMIT
Get only the first 3/4/7/n customers (rows)
```
SELECT * FROM Customers LIMIT 3;
```
Useful to limit customers per page (eg page1:1-3, page2:4-6 etc)
```
SELECT * FROM Customers
LIMIT 6,3; --- afiseaza maxim 3 persoane si adauga un offset de 6 persoane (skip la primii 6)
```

---

## <a name="INNERJOIN"></a>INNER JOIN
```
SELECT order_id, orders.customer_id, first_name, last_name
FROM orders
JOIN customers ON orders.customer_id = customers.customer_id;
```
```
SELECT * FROM Sectii
JOIN Pacienti ON Sectii.IdSectie = Paciennti.IdSectie;

/* or */

SELECT * FROM Pacienti
JOIN Sectii ON Pacienti.IdSectie = Sectii.IdSectie;
```
| IdPacient | Nume        | Prenume | IdSectie | IdSectie | Nume | Buget |
| --------- | ----------- | ------- | -------- | -------- | --- | ----- |
| 1   | Popescu    | Ana | **1** | **1** | s1 | 5500
| 2   | Munteanu    | Alex | **3** | **3** | s3 | 6000
| 3   |     Dobre   | Cosmin | **2** | **2** | s2 | 5200 |
| 4   |     Freeman | John | **1** | **1** | s1 | 4000 |

If we use ALIAS (Obs: Daca o coloana are acelasi nume in cealalta tabelă, trebuie specificata din care tabela faci SELECT (ce coloană afisezi):
```
SELECT p.Nume, Prenume, p.IdSectie, s.Nume, Buget
FROM pacienti p
JOIN Sectii s on p.IdSectie = s.IdSectie;
```
**OBS:** Putem face JOIN si la tabele care se afla in _alte baze de date_ (different database)
```
SELECT * FROM order_items oi
JOIN another_database.products p ON oi.product_id = p.product_id;
```

### <a name="INNERJOIN_MULTIPLE"></a>INNER JOIN: Multiple Tables
```
SELECT * ---o.order_id, o.order_date, c.name, os.name AS status
FROM orders o
JOIN customers c
  ON o.customer_id = c.customer_id
JOIN order_statuses os
  ON o.status = OS.order_status_id;
```

### <a name="INNERJOIN_COMPOUND"></a>COMPOUND JOIN
**INNER JOIN for COMPOSITE PRIMARY KEY --- chei primare compuse, care conțin cel puțin 2 atribute**
| order_items |
| ---------- |
| (PK) order_id |
| (PK) product_id |
| quantity |
| unit_price |
```
SELECT * FROM order_items oi
JOIN order_items_notes oin
  ON oi.order_id = oin.order_id
  AND oi.product_id = oin.product_id;
```

## <a name="OUTTERJOIN"></a>OUTTER JOIN
Util pentru a afișa datele care au NULL la atributul cheie străină (in al doilea tabel)
```
SELECT *
FROM Customers c
LEFT JOIN Orders o
  ON c.customers_id = o.customers_id
ORDER BY c.customers_id;
```
| id | name     | order_id |
|----|----------|----------|
| 1  | Innes    | 7        |
| 2  | Freddy   | NULL     |
| 3  | Carolina | 2        |
| 4  | Elka     | NULL     |

```
SELECT
  c.customer_id
  c.first_name
  o.order_id
FROM customers c
LEFT JOIN orders o
  ON c.customer_id = o.customer_id;
  
/* este echivalent cu */

SELECT
  c.customer_id
  c.first_name
  o.order_id
FROM orders o
RIGHT JOIN customers c
  ON c.customer_id = o.customer_id;
```
OBS: Este de evitat folosirea RIGHT JOIN in special pentru mai mult de 2 tabele deoarece creeaza confuzie.

## <a name="USING"></a>USING
Folosit pentru 2 coloane cu exact acelasi nume (eg: customer_id) - coloane din 2 tabele diferite
```
SELECT * FROM Customers c
LEFT JOIN Orders o
  USING(customer_id);
```

```
SELECT p.Nume, Prenume, P.IdSectie, s.Nume AS Nume_Sectie, Buget
FROM pacienti p
JOIN sectii s
  USING(IdSectie);
```
OBS: USING functioneaza si pentru chei compuse:
```
SELECT *
FROM order_items oi
JOIN order_items_notes oin
  USING(order_id, product_id);
```
| order_items     |                  | order_items_notes |
|-----------------|------------------|-------------------|
| (PK) order_id   |                  | (PK) oin_id       |
| (PK) product_id |                  | attr_etc          |
| price           |                  | (FK) order_id     |
|                 |                  | (FK) product_id   |

---

## <a name="UNIONS"></a>UNIONS
Combine rows from multiple tables
```
SELECT Nume, Prenume
FROM Pacienti
UNION
SELECT Nume, Buget
FROM Sectii;
```
OBS 1: Numarul de coloane returnate din prima tabela trebuie sa fie egal cu numarul de coloane din a doua tabela
```
SELECT Nume as Full_Name
FROM Shippers
UNION
SELECT Name
FROM Customers;
```
OBS 2: Numaele coloanei afisate va aparea ca numele coloanei primei tabele

---

## <a name="INSERT"></a>INSERT INTO
OBS: Ordinea contează (The order of attributes matters)
```
INSERT INTO Customers(first_name, last_name, birth_date, address, city)
  VALUES('John', 'Smith', '1990-01-01', 'address1', 'city2');
```
```
INSERT INTO Pacienti(Nume, Prenume, Judet, IdSectie)
  VALUES('Poepescu', 'Ion', 'Bacau', 3);
```

### <a name="INSERT_MULTIPLE"></a>INSERT MULTIPLE VALUES INTO A TABLE
```
INSERT INTO pacienti(Nume, Prenume, Judet)
VALUES('Horia', 'Alex', 'Timisoara'),
      ('Popa', 'Raluca', 'Olt');
```

### <a name="INSERT_MULTIPLE2"></a>INSERT DATAINTO MULTIPLE TABLES
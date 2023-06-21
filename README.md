# esercizi-t-sql

AdventureWorks2014 è un database di esempio che è stato rilasciato da Microsoft per SQL Server. È un database di esempio complesso e ben strutturato per un'azienda di vendita di biciclette e accessori.


Di seguito sono riportati alcuni esercizi che puoi provare a fare utilizzando il database AdventureWorks2014.

https://learn.microsoft.com/it-it/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms



TIPOLOGIE DI JOIN :

https://database.guide/sql-joins-tutorial/



##     Esercizio 1: Seleziona tutti i nomi dei prodotti dalla tabella Production.Product.


```
SELECT Name
FROM Production.Product;


```


##     Esercizio 2: Seleziona i nomi e gli indirizzi email di tutti i dipendenti dalla tabella HumanResources.Employee e Person.EmailAddress.


```
SELECT E.BusinessEntityID, P.FirstName, P.LastName, EA.EmailAddress
FROM HumanResources.Employee AS E
JOIN Person.Person AS P ON E.BusinessEntityID = P.BusinessEntityID
JOIN Person.EmailAddress AS EA ON E.BusinessEntityID = EA.BusinessEntityID;


```


## Esercizio 3: Seleziona i nomi dei prodotti e i nomi dei fornitori associati.

```
SELECT P.Name AS ProductName, V.Name AS VendorName
FROM Production.Product AS P
JOIN Purchasing.ProductVendor AS PV ON P.ProductID = PV.ProductID
JOIN Purchasing.Vendor AS V ON PV.VendorID = V.BusinessEntityID;
```

## Esercizio 4: Seleziona i clienti che hanno effettuato un ordine dopo una certa data (ad es. '2014-01-01').


```
SELECT DISTINCT C.CustomerID
FROM Sales.Customer AS C
JOIN Sales.SalesOrderHeader AS SO ON C.CustomerID = SO.CustomerID
WHERE SO.OrderDate > '2014-01-01';


```


## Esercizio 5: Seleziona i prodotti che hanno un prezzo di listino superiore a un certo valore (ad es. 1000).

```


SELECT Name, ListPrice
FROM Production.Product
WHERE ListPrice > 1000;

```


##  Esercizio 6: Seleziona i primi 10 clienti ordinati per cognome in ordine alfabetico.


```
SELECT TOP 10 FirstName, LastName
FROM Person.Person
WHERE BusinessEntityID IN (
    SELECT BusinessEntityID
    FROM Sales.Customer
)
ORDER BY LastName;


```

## Esercizio 7: Seleziona le categorie di prodotto e il numero di prodotti in ciascuna categoria.

```

SELECT PC.Name AS Category, COUNT(*) AS NumberOfProducts
FROM Production.ProductCategory AS PC
JOIN Production.ProductSubcategory AS PS ON PC.ProductCategoryID = PS.ProductCategoryID
JOIN Production.Product AS P ON PS.ProductSubcategoryID = P.ProductSubcategoryID
GROUP BY PC.Name;


```


##   Esercizio 8: Seleziona il totale degli ordini effettuati per ogni anno.

```
SELECT YEAR(OrderDate) AS OrderYear, COUNT(*) AS TotalOrders
FROM Sales.SalesOrderHeader
GROUP BY YEAR(OrderDate);
```

##     Esercizio 9: Seleziona i dipendenti che hanno un cognome che inizia con la lettera 'B'.

```
SELECT FirstName, LastName
FROM Person.Person
WHERE BusinessEntityID IN (
    SELECT BusinessEntityID
    FROM HumanResources.Employee
)
AND LastName LIKE 'B%';


```

## Esercizio 10: Seleziona i prodotti che sono stati venduti in quantità superiore a 10 in un unico ordine.
```
SELECT ProductID, Name
FROM Production.Product
WHERE ProductID IN (
    SELECT ProductID
    FROM Sales.SalesOrderDetail
    WHERE OrderQty > 10
);

```

# ROLLBACK

In informatica, il termine "rollback" si riferisce al processo di annullamento di un insieme di transazioni o modifiche al database e di ripristino dello stato del database a un punto precedente nel tempo. Il rollback è una caratteristica fondamentale dei sistemi di gestione dei database che garantisce l'integrità dei dati in caso di errori o fallimenti del sistema.

Ecco come funziona il rollback, in genere, in SQL:

    Inizi una transazione.
    Esegui una serie di operazioni sul database (come inserimento, aggiornamento o eliminazione).
    Se tutto va bene, puoi eseguire un commit per rendere le modifiche permanenti.
    Se qualcosa va storto, esegui un rollback per annullare tutte le modifiche effettuate durante la transazione.



### Esempio:

Immaginiamo di avere una tabella chiamata Accounts con due colonne: AccountID e Balance.


```
CREATE TABLE Accounts (
    AccountID INT PRIMARY KEY,
    Balance DECIMAL(10, 2)
);


```


Inseriamo un paio di righe:

```
INSERT INTO Accounts VALUES (1, 1000.00);
INSERT INTO Accounts VALUES (2, 2000.00);
```


Ora eseguiamo una transazione che trasferisce denaro da un account all'altro, ma decidiamo di annullarla prima di renderla permanente:
```
BEGIN TRANSACTION;

UPDATE Accounts SET Balance = Balance - 100 WHERE AccountID = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE AccountID = 2;

-- Immagina che qualcosa sia andato storto (ad es. errore logico, fallimento del sistema, ecc.)

ROLLBACK;
```



** ESERCIZI CON ROLLBACK **

Esercizio 1: Prova ad aggiungere un nuovo prodotto alla tabella Production.Product e poi esegui un rollback per annullare l'operazione.


```
BEGIN TRANSACTION;

INSERT INTO Production.Product (Name, ProductNumber, StandardCost, ListPrice, ProductLine)
VALUES ('New Bike', 'BK-NEW-01', 250, 500, 'M');

ROLLBACK;
```


Esercizio 2: Prova ad aggiornare il prezzo dei prodotti in una categoria specifica, ma poi annulla l'operazione con un rollback.
```
BEGIN TRANSACTION;

UPDATE Production.Product
SET ListPrice = ListPrice * 1.10
WHERE ProductSubcategoryID IN (
    SELECT ProductSubcategoryID
    FROM Production.ProductSubcategory
    WHERE Name = 'Mountain Bikes'
);

ROLLBACK;

```


Esercizio 3: Prova a trasferire un prodotto da un magazzino ad un altro e poi annulla l'operazione con un rollback.


```
BEGIN TRANSACTION;

DECLARE @ProductID INT = 680;
DECLARE @FromLocationID INT = 1;
DECLARE @ToLocationID INT = 5;
DECLARE @Quantity INT = 10;

UPDATE Production.ProductInventory
SET Quantity = Quantity - @Quantity
WHERE ProductID = @ProductID AND LocationID = @FromLocationID;

UPDATE Production.ProductInventory
SET Quantity = Quantity + @Quantity
WHERE ProductID = @ProductID AND LocationID = @ToLocationID;

ROLLBACK;
```




Esercizio 4: Prova ad inserire un nuovo ordine e le relative righe dell'ordine, ma poi esegui un rollback se l'ordine supera un certo importo.


```
BEGIN TRANSACTION;

DECLARE @OrderTotal DECIMAL(10, 2);

INSERT INTO Sales.SalesOrderHeader (CustomerID, OrderDate, DueDate, ShipDate, SalesOrderNumber, SubTotal, TaxAmt)
VALUES (1, GETDATE(), DATEADD(day, 7, GETDATE()), DATEADD(day, 2, GETDATE()), 'SO99999', 1000, 100);

SET @OrderTotal = 1000;

IF @OrderTotal > 500
BEGIN
    ROLLBACK;
END
ELSE
BEGIN
    INSERT INTO Sales.SalesOrderDetail (SalesOrderID, ProductID, OrderQty, UnitPrice)
    VALUES (SCOPE_IDENTITY(), 680, 2, 20);

    COMMIT;
END


```


Esercizio 5: Prova a eliminare un fornitore dalla tabella Purchasing.Vendor e poi esegui un rollback per annullare l'operazione.


```
BEGIN TRANSACTION;

DELETE FROM Purchasing.Vendor
WHERE BusinessEntityID = 100;

ROLLBACK;
```


# ESERCIZI CON GROUP BY 

    Esercizio 1: Trova il numero di prodotti in ogni categoria.


```

SELECT pc.Name AS CategoryName, COUNT(*) AS NumberOfProducts
FROM Production.ProductCategory AS pc
JOIN Production.ProductSubcategory AS psc ON pc.ProductCategoryID = psc.ProductCategoryID
JOIN Production.Product AS p ON psc.ProductSubcategoryID = p.ProductSubcategoryID
GROUP BY pc.Name;


```


Esercizio 2: Calcola il costo medio dei prodotti per ogni categoria che abbia almeno 5 prodotti.

```
SELECT pc.Name AS CategoryName, AVG(p.StandardCost) AS AverageCost
FROM Production.ProductCategory AS pc
JOIN Production.ProductSubcategory AS psc ON pc.ProductCategoryID = psc.ProductCategoryID
JOIN Production.Product AS p ON psc.ProductSubcategoryID = p.ProductSubcategoryID
GROUP BY pc.Name
HAVING COUNT(*) >= 5;
```


Esercizio 3: Trova il totale dei prezzi di listino per prodotti in ogni colore.
```
SELECT Color, SUM(ListPrice) AS TotalListPrice
FROM Production.Product
GROUP BY Color;

```


```
Esercizio 4: Trova il numero di ordini effettuati ogni anno.


SELECT YEAR(OrderDate) AS OrderYear, COUNT(*) AS NumberOfOrders
FROM Sales.SalesOrderHeader
GROUP BY YEAR(OrderDate);


```


Esercizio 5: Trova il totale delle quantità vendute per ogni prodotto.

```
SELECT ProductID, SUM(OrderQty) AS TotalQuantity
FROM Sales.SalesOrderDetail
GROUP BY ProductID;


```



Esercizio 6: Trova il totale venduto per ogni venditore che abbia venduto più di 100 articoli.


```

SELECT SalesPersonID, SUM(TotalDue) AS TotalSales
FROM Sales.SalesOrderHeader
GROUP BY SalesPersonID
HAVING SUM(TotalDue) > 100;


```




Esercizio 7: Trova il totale delle ore lavorate per ogni tipo di operazione di lavoro.


```
SELECT OperationSequence, SUM(ActualResourceHrs) AS TotalHours
FROM Production.WorkOrderRouting
GROUP BY OperationSequence;

```


Esercizio 8: Conta il numero di clienti per ogni tipo di cliente, ma solo per i tipi che hanno più di 10 clienti.

```

SELECT CustomerType, COUNT(*) AS NumberOfCustomers
FROM Sales.Customer
GROUP BY CustomerType
HAVING COUNT(*) > 10;


```



    Esercizio 9: Trova il numero di dipendenti in ogni ruolo di job.

```
SELECT JobTitle, COUNT(*) AS NumberOfEmployees
FROM HumanResources.Employee
GROUP BY JobTitle;
```


    Esercizio 10: Trova la media dei saldi dei conti (Account Receivable) per ogni cliente, ma solo per i clienti che hanno un saldo medio superiore a 1000.
```
SELECT CustomerID, AVG(AccountReceivable) AS AverageBalance
FROM Sales.CustomerAccount
GROUP BY CustomerID
HAVING AVG(AccountReceivable) > 1000;
```


# SUBQUERY :


Le subquery, o query nidificate, sono query SQL che sono inserite all'interno di un'altra query SQL. Una subquery può essere utilizzata in diverse parti della query principale come nelle clausole SELECT, FROM, WHERE, o HAVING.

Le subquery sono utilizzate per diversi scopi, come:

    Filtrare risultati basati su dati aggregati.
    Recuperare dati per un confronto che non può essere fatto direttamente.
    Semplificare le query complesse decomponendole in parti più piccole e gestibili.



## Tipi di Subquery

    Subquery Scalar: Ritornano un singolo valore. Queste possono essere utilizzate in qualsiasi luogo in cui ti aspetti un valore singolo, come in una clausola di confronto.

```

SELECT * FROM Products
WHERE price > (
    SELECT AVG(price) FROM Products
);

```


Subquery in clausola FROM: Puoi utilizzare una subquery come una tabella in una clausola FROM.

```
SELECT AVG(Price)
FROM (
    SELECT Price FROM Products WHERE Category = 'Bicycles'
) AS BicyclePrices;
```

Subquery in clausola IN: Queste subquery ritornano un set di valori che vengono poi utilizzati per il filtraggio nella query principale.


```
SELECT * FROM Orders
WHERE CustomerID IN (
    SELECT CustomerID FROM Customers WHERE Country = 'USA'
);

```


Subquery correlate: Sono subquery che fanno riferimento a una o più colonne della query esterna. Queste sono eseguite una volta per ogni riga processata dalla query esterna.


```
SELECT e.Name, e.Salary
FROM Employees e
WHERE e.Salary > (
    SELECT AVG(e2.Salary)
    FROM Employees e2
    WHERE e.DepartmentID = e2.DepartmentID
);


```

Strategie per utilizzare le Subquery

Scomposizione di problemi complessi: Utilizza subquery per scomporre problemi di query complessi in passi più piccoli e gestibili.
Migliora la leggibilità: Utilizza subquery per migliorare la leggibilità separando logicamente diverse parti della query.

Utilizza subquery correlate con cautela: Le subquery correlate possono avere gravi impatti sulle prestazioni, in quanto vengono eseguite per ogni riga nella query esterna.

Evita subquery non necessarie: Se una subquery può essere riscritta utilizzando JOINs o altre tecniche senza impatto sulla leggibilità o la logica, allora generalmente dovresti preferire la versione senza subquery per motivi di prestazioni.

Test ed ottimizzazione: Esegui test sulle prestazioni delle tue query nidificate e ottimizzale se necessario. A volte, piccoli cambiamenti nella struttura di una subquery possono portare a miglioramenti significativi delle prestazioni.


Seleziona i prodotti che sono stati venduti in quantità superiore a 10 in un unico ordine.

```
SELECT ProductID, Name
FROM Production.Product
WHERE ProductID IN (
    SELECT ProductID
    FROM Sales.SalesOrderDetail
    WHERE OrderQty > 10
);
```



Trova i nomi dei prodotti il cui prezzo è superiore alla media dei prezzi.

```
SELECT Name, ListPrice
FROM Production.Product
WHERE ListPrice > (SELECT AVG(ListPrice) FROM Production.Product);
```


Commento: La subquery calcola la media dei prezzi dei prodotti, e la query principale seleziona i nomi dei prodotti con un prezzo superiore alla media.


Trova tutti i clienti che hanno fatto ordini nel 2013.


```
SELECT DISTINCT CustomerID
FROM Sales.SalesOrderHeader
WHERE YEAR(OrderDate) = 2013
AND CustomerID IN (SELECT CustomerID FROM Sales.Customer);
```


Commento: La subquery seleziona tutti i CustomerID dalla tabella Customer, mentre la query principale seleziona i clienti che hanno effettuato ordini nel 2013.



Trova i dettagli dei prodotti che sono stati venduti in quantità maggiore di 100 unità in un singolo ordine.



```
SELECT *
FROM Production.Product
WHERE ProductID IN (SELECT ProductID FROM Sales.SalesOrderDetail WHERE OrderQty > 100);
```

Commento: La subquery seleziona i ProductID che sono stati venduti in quantità superiore a 100, mentre la query principale estrae i dettagli di quei prodotti.



Trova il totale delle vendite per ogni categoria di prodotto.

```
SELECT ProductCategoryID, SUM(TotalDue)
FROM (SELECT Product.ProductCategoryID, SalesOrderDetail.LineTotal AS TotalDue
      FROM Sales.SalesOrderDetail SalesOrderDetail
      JOIN Production.Product Product
      ON SalesOrderDetail.ProductID = Product.ProductID) AS Subquery
GROUP BY ProductCategoryID;
```

Commento: Questo utilizza una subquery come tabella derivata per calcolare il totale delle vendite per ogni categoria di prodotto.


Trova i clienti che non hanno fatto ordini nel 2014.

```
SELECT CustomerID
FROM Sales.Customer
WHERE CustomerID NOT IN (SELECT DISTINCT CustomerID
                         FROM Sales.SalesOrderHeader
                         WHERE YEAR(OrderDate) = 2014);
```

Commento: La subquery seleziona i CustomerID che hanno fatto ordini nel 2014, e la query principale seleziona quelli che non sono in quella lista.



Trova il prodotto più costoso in ogni categoria.

```
SELECT ProductCategoryID, MAX(ListPrice) AS MaxPrice
FROM Production.Product
WHERE ListPrice IN (SELECT MAX(ListPrice)
                    FROM Production.Product
                    GROUP BY ProductCategoryID)
GROUP BY ProductCategoryID;

```

Commento: La subquery seleziona il prezzo più alto per ogni categoria, e la query principale estrae il prodotto con quel prezzo in ogni categoria.


Trova i fornitori che forniscono almeno 3 prodotti.


```
SELECT ProductVendorID
FROM Purchasing.ProductVendor
WHERE ProductVendorID IN (SELECT ProductVendorID
                          FROM Purchasing.ProductVendor
                          GROUP BY ProductVendorID
                          HAVING COUNT(*) >= 3);

```



Commento: La subquery seleziona i fornitori che forniscono almeno 3 prodotti, e la query principale estrae i dettagli di questi fornitori.



Trova il totale delle vendite per i clienti che hanno effettuato almeno 2 ordini.

```
SELECT CustomerID, SUM(TotalDue)
FROM Sales.SalesOrderHeader
WHERE CustomerID IN (SELECT CustomerID
                     FROM Sales.SalesOrderHeader
                     GROUP BY CustomerID
                     HAVING COUNT(*) >= 2)
GROUP BY CustomerID;
```



Commento: La subquery seleziona i CustomerID che hanno effettuato almeno 2 ordini, mentre la query principale calcola il totale delle vendite per questi clienti.

Trova il nome della categoria a cui appartiene il prodotto più costoso.


```
SELECT Name
FROM Production.ProductCategory
WHERE ProductCategoryID = (SELECT ProductCategoryID
                           FROM Production.Product
                           WHERE ListPrice = (SELECT MAX(ListPrice)
                                              FROM Production.Product));
```

Commento: Questo esercizio utilizza due subquery annidate. La prima subquery più interna trova il prezzo più alto tra tutti i prodotti, la seconda trova la categoria di quel prodotto e la query principale estrae il nome della categoria.



Trova i nomi dei dipendenti che hanno lo stesso titolo di lavoro di 'Production Technician - WC60'.


```
SELECT FirstName, LastName
FROM HumanResources.Employee
JOIN Person.Person ON HumanResources.Employee.BusinessEntityID = Person.Person.BusinessEntityID
WHERE JobTitle = (SELECT JobTitle FROM HumanResources.Employee WHERE JobTitle = 'Production Technician - WC60');

```

Commento: La subquery trova il titolo di lavoro 'Production Technician - WC60', e la query principale seleziona i nomi dei dipendenti con quel titolo.



Trova i fornitori che forniscono solo prodotti di una specifica categoria.

```
SELECT VendorID
FROM Purchasing.ProductVendor
WHERE VendorID NOT IN (SELECT VendorID
                       FROM Purchasing.ProductVendor
                       JOIN Production.Product ON Purchasing.ProductVendor.ProductID = Production.Product.ProductID
                       GROUP BY VendorID
                       HAVING COUNT(DISTINCT ProductCategoryID) > 1);
```

Commento: La subquery seleziona i fornitori che forniscono prodotti di più di una categoria, mentre la query principale esclude questi fornitori.





Trova le categorie di prodotti che hanno un prezzo medio superiore a $100.

```
SELECT ProductCategoryID, AVG(ListPrice) AS AvgPrice
FROM Production.Product
GROUP BY ProductCategoryID
HAVING AVG(ListPrice) > 100
AND ProductCategoryID IN (SELECT ProductCategoryID FROM Production.Product);

```
Commento: La subquery seleziona tutte le categorie di prodotti, e la query principale calcola il prezzo medio per ogni categoria e seleziona quelle con un prezzo medio superiore a $100.



Trova il cliente che ha effettuato il maggior numero di ordini.


```
SELECT CustomerID
FROM Sales.SalesOrderHeader
GROUP BY CustomerID
HAVING COUNT(*) = (SELECT MAX(OrdersCount)
                   FROM (SELECT COUNT(*) AS OrdersCount
                         FROM Sales.SalesOrderHeader
                         GROUP BY CustomerID) AS Subquery);

```


Commento: La subquery annidata conta il numero di ordini per ogni cliente, e la subquery esterna trova il massimo di questi conteggi. La query principale seleziona il cliente con quel numero di ordini.



Trova il mese in cui ci sono state le maggiori vendite totali.

```
SELECT MONTH(OrderDate) AS Month, SUM(TotalDue) AS TotalSales
FROM Sales.SalesOrderHeader
GROUP BY MONTH(OrderDate)
HAVING SUM(TotalDue) = (SELECT MAX(Sales)
                        FROM (SELECT SUM(TotalDue) AS Sales
                              FROM Sales.SalesOrderHeader
                              GROUP BY MONTH(OrderDate)) AS Subquery);
```

Commento: Simile all'esercizio precedente, ma qui stiamo cercando il mese con le maggiori vendite totali.



Trova i prodotti che non sono mai stati venduti.

```
SELECT Name
FROM Production.Product
WHERE ProductID NOT IN (SELECT DISTINCT ProductID FROM Sales.SalesOrderDetail);
```
Commento: La subquery seleziona tutti i ProductID che sono stati venduti, e la query principale seleziona i prodotti che non sono in quella lista.


    Trova i dipendenti che non hanno un manager nello stesso dipartimento.













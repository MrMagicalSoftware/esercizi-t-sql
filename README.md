# esercizi-t-sql

AdventureWorks2014 è un database di esempio che è stato rilasciato da Microsoft per SQL Server. È un database di esempio complesso e ben strutturato per un'azienda di vendita di biciclette e accessori.


Di seguito sono riportati alcuni esercizi che puoi provare a fare utilizzando il database AdventureWorks2014.

https://learn.microsoft.com/it-it/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms

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













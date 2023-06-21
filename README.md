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


Esercizio 5: Seleziona i prodotti che hanno un prezzo di listino superiore a un certo valore (ad es. 1000).

```


SELECT Name, ListPrice
FROM Production.Product
WHERE ListPrice > 1000;

```










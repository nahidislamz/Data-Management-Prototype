# Data Management Prototype

**Coursework Aim:** 
*To demonstrate understanding of the module subject matter and the ability to apply it, including critical analysis and judgement. This coursework is based on individual submissions. Design, Implement, Test, Report on a prototype data warehouse.* 

 - Using Kimball’s Dimensional Modelling Technique design a suitable Star Schema for the specified data sets. 
- Then using either MS SQL Server with T-SQL or SSIS the necessary ETL operations and 4 query tasks (programming functionality) must be implemented. 
- Complete with a dashboard built utilising the 4 query tasks in Tableau or Power BI. 

# Subject Knowledge Design 
*Use Dimensional Modelling Technique to design a schema for a prototype data warehouse, using the data on HM Home Office spending over £25,000*

![Schema Design](https://github.com/nahidislamz/Data-Management-Prototype/blob/main/images/ERD.png)

## Critical Analysis – Transforming design to implementation

![Merge_CSV](https://github.com/nahidislamz/Data-Management-Prototype/blob/main/images/merging_data.png)

![Data_Transformation](https://github.com/nahidislamz/Data-Management-Prototype/blob/main/images/Screenshot%20%2847%29.png)

# Query - 1
### a. Calculate the top three payment types and the total amount spent on these payment types and in each of the four quarters April June, July-September, October-December, and January-March.
```sql
SELECT TOP 3  ExpenseType , Spend AS TotalSpend , 'APRIL - JUNE' AS 'Duration'  
FROM FactTable AS fact
left join DateTable AS dates ON fact.Date_ID = dates.Date_ID
left join ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID WHERE dates between '2017-04-01'and '2017-06-30'
UNION
SELECT TOP 3  ExpenseType , Spend AS TotalSpend , 'JULY - SEPTEMBER' AS 'Duration'  
FROM FactTable AS fact
left join DateTable AS dates ON fact.Date_ID = dates.Date_ID
left join ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID WHERE dates between '2017-07-01' and  '2017-09-30' 
UNION
SELECT TOP 3  ExpenseType , Spend AS TotalSpend , 'OCTOBER - DECEMBER' AS 'Duration'  
FROM FactTable AS fact
left join DateTable AS dates ON fact.Date_ID = dates.Date_ID
left join ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID WHERE dates between '2017-10-01' and '2017-12-31' 
UNION
SELECT TOP 3  ExpenseType , Spend AS TotalSpend , 'JANUARY - MARCH' AS 'Duration'  
FROM FactTable AS fact
left join DateTable AS dates ON fact.Date_ID = dates.Date_ID
left join ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID WHERE dates between '2018-01-01' and '2018-03-31'  
ORDER BY TotalSpend DESC
```

# Query  - 2
#### b. Calculate the top four expense areas and the totals amount spent on these areas for the year and for each of the two half-years April - Sept and Oct - Mar
```sql
SELECT TOP 3  ExpenseArea , Spend AS TotalSpend , 'YEAR' AS 'Duration'  
FROM FactTable AS fact
left join DateTable AS dates ON fact.Date_ID = dates.Date_ID
left join ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID
UNION
SELECT TOP 3  ExpenseArea , Spend AS TotalSpend , 'APRIL - SEPTEMBER' AS 'Duration'  
FROM FactTable AS fact
left join DateTable AS dates ON fact.Date_ID = dates.Date_ID
left join ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID WHERE dates between '2017-04-01' and  '2017-09-30' 
UNION
SELECT TOP 3  ExpenseArea , Spend AS TotalSpend , 'OCTOBER - MARCH' AS 'Duration'  
FROM FactTable AS fact
left join DateTable AS dates ON fact.Date_ID = dates.Date_ID
left join ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID WHERE dates between '2017-10-01' and '2018-03-31' 
ORDER BY TotalSpend DESC 
```
# Query - 3
#### For each quarter of the year rank the top 10 Suppliers by total net spend made to them by the home office. Clearly indicate the change in rank for each quarter The rankings must be in ascending order.
```sql
SELECT TOP 10 SupplierName, SUM(Spend) AS Spending ,     
RANK() OVER(ORDER BY SUM(Spend) DESC) Q , 0 AS Q2 , 0 AS Q3 , 0 AS Q4  
FROM FactTable AS fact
INNER JOIN DateTable AS dates ON fact.Date_ID = dates.Date_ID
INNER JOIN ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID
INNER JOIN SupplierTable AS supplier ON fact.Supplier_ID = supplier.Supplier_ID
WHERE dates BETWEEN '2017-04-01' AND '2017-06-30' GROUP BY SupplierName 
UNION
SELECT TOP 10 SupplierName, SUM(Spend) AS Spending , 0 AS Q1,
RANK() OVER(ORDER BY SUM(Spend) DESC) Q2 , 0 AS Q3 , 0 AS Q4  
FROM FactTable AS fact
INNER JOIN DateTable AS dates ON fact.Date_ID = dates.Date_ID
INNER JOIN ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID
INNER JOIN SupplierTable AS supplier ON fact.Supplier_ID = supplier.Supplier_ID
WHERE dates BETWEEN '2017-07-01' AND '2017-09-30' GROUP BY SupplierName
UNION
SELECT TOP 10 SupplierName, SUM(Spend) AS Spending , 0 AS Q1, 0 AS Q2 ,
RANK() OVER(ORDER BY SUM(Spend) DESC) Q3 , 0 AS Q4  
FROM FactTable AS fact
INNER JOIN DateTable AS dates ON fact.Date_ID = dates.Date_ID
INNER JOIN ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID
INNER JOIN SupplierTable AS supplier ON fact.Supplier_ID = supplier.Supplier_ID
WHERE dates BETWEEN '2017-10-01' AND '2017-12-31' GROUP BY SupplierName
UNION
SELECT TOP 10 SupplierName, SUM(Spend) AS Spending , 0 AS Q1, 0 AS Q2 , 0 AS Q3, 
RANK() OVER(ORDER BY SUM(Spend) DESC) Q4
FROM FactTable AS fact
INNER JOIN DateTable AS dates ON fact.Date_ID = dates.Date_ID
INNER JOIN ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID
INNER JOIN SupplierTable AS supplier ON fact.Supplier_ID = supplier.Supplier_ID
WHERE dates BETWEEN '2018-01-01' AND '2018-03-31' GROUP BY SupplierName ORDER BY Spending ASC
```
# Query -4
#### d. Create a fourth complex query utilizing a time related hierarchy of your own design that demonstrates the full complexity of your understanding in terms of analytic query writing.*/
```sql
SELECT TOP 3 Entity, AVG(Spend) AS AverageSpend , TransactionNumber, 'APRIL - SEPTEBER' AS 'Duration'  
FROM FactTable AS fact
left join DateTable AS dates ON fact.Date_ID = dates.Date_ID
left join ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID
left join EntityTable as entity on fact.[Entity_ID] = entity.[Entity_ID]
left join TransactionTable AS T ON fact.Transaction_ID = T.Transaction_ID WHERE dates between '2017-04-01' and  '2017-09-30'
GROUP BY Entity,TransactionNumber
UNION
SELECT TOP 3 Entity, AVG(Spend) AS AverageSpend , TransactionNumber,  'OCTOBER - MARCH' AS 'Duration'  
FROM FactTable AS fact
left join DateTable AS dates ON fact.Date_ID = dates.Date_ID
left join ExpenseTable AS expense ON fact.Expense_ID = expense.Expense_ID
left join EntityTable as entity on fact.[Entity_ID] = entity.[Entity_ID]
left join TransactionTable AS T ON fact.Transaction_ID = T.Transaction_ID WHERE dates between '2017-10-01' and '2018-03-31' 
GROUP BY Entity,TransactionNumber
ORDER BY AverageSpend DESC 
```

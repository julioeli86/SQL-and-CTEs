# SQL Subqueries, Views, and Common Table Expressions (CTEs)


A total of six SQL tables were created using the SQL Server Management Studio (SQL-SMS) and following the Entity Relationship Diagram (ERD) provided in Fig.- 1

![image](ERD-Mod-10-002.jpg)

The below SELECT commands were executed on the SQL Server Management Studio.

```
SELECT * FROM yEmp; 
SELECT * FROM yProd; 
SELECT * FROM yOrd; 
SELECT * FROM yOrderLine; 
SELECT * FROM yCust; 
SELECT * FROM yJobTitle;
```

The following SELECT commands were executed to identify which customers have orders in the order table:

```
SELECT * 
FROM yCust 
WHERE custid IN 
        (SELECT custid 
        FROM yOrd)
```

The below SELECT commands helps us to identify which customers do not have orders in the order table:

```
SELECT * 
FROM yCust 
WHERE custid NOT IN 
        (SELECT custid 
        FROM yOrd)
```

The below commands provide the orders that are not affiliated with an employee. In addition, the commands check for empid with null values.

```
SELECT * 
FROM yOrd 
WHERE empid NOT IN 
        (SELECT empid 
        FROM yEmp) 
OR empid is null
```

## FIRST ENTRY
To identify the employees with a pay rate that is less than or equal to the average employee pay rate the following commands were executed:

```
SELECT	EmpID,
		EmpName,
		PayRate,
		jt.Title,
		(SELECT AVG (payrate)
		FROM	yEmp) AveragePayRate
FROM	yemp	emp
INNER JOIN  yJobtitle jt
ON	emp.jobtitleid	= jt.jobtitleID
WHERE	payrate <= 
	(SELECT AVG (payrate)
	FROM	yemp)
ORDER BY emp.empid
```

The results from the First Entry are shown in Fig. 2 below.

![image](BAN-702-FIG-002.jpg)

## SECOND ENTRY
Modify the query from the First Entry to include the max and min pay rate for all employees from the SELECT list.

```
SELECT	EmpID,
		EmpName,
		PayRate,
		jt.Title,
		(SELECT AVG (payrate)
		FROM	yEmp) AveragePayRate,
		(SELECT MAX(payrate)
		FROM yEmp) MaximumPayRate,
		(SELECT MIN(payrate)
		FROM yEmp)	MinimumPayRate
FROM	yemp	emp
INNER JOIN  yJobtitle jt
ON	emp.jobtitleid	= jt.jobtitleID
WHERE	payrate <= 
	(SELECT AVG (payrate)
	FROM	yemp)
ORDER BY emp.empid
```

The results from the Second Entry are shown in Fig. 3 below.

![image](BAN-702-FIG-003.jpg)

## THIRD ENTRY
The Third Entry focuses on showing the use of the correlated subquery. The following SELECT commands identify those employees that have a higher payrate than the payrate for their job title.

```
SELECT EmpID,
	EmpName,
	PayRate,
	jt.Title
FROM yemp emp
INNER JOIN yjobtitle jt
ON emp.jobtitleid = jt.jobtitleid
WHERE payrate >
	(SELECT AVG(payrate)
	FROM yemp empINNER
	WHERE empinner.jobtitleid = emp.jobtitleid)
ORDER BY emp.empid
```

The following query adds the average pay rate for the corresponding job title.

```
SELECT EmpID,
	EmpName,
	PayRate,
	jt.Title,
	(SELECT AVG(payrate)
	FROM yemp empSELECT
	WHERE empSELECT.jobtitleid = emp.jobtitleid) AveragePayRate
FROM yemp emp
INNER JOIN yjobtitle jt
ON emp.jobtitleid = jt.jobtitleid
WHERE payrate >
	(SELECT AVG(payrate)
	FROM yemp empINNER
	WHERE empinner.jobtitleid = emp.jobtitleid)
ORDER BY emp.empid
```

The following query provides the employees with a payrate that is less than or equal to the average payrate for their job title.

```
SELECT	EmpID,
		EmpName,
		PayRate,
		jt.Title,
		(SELECT AVG(payrate)
		FROM	yemp empSELECT
		WHERE	empSELECT.jobtitleid = emp.jobtitleid)	AveragePayRate
FROM	yemp emp
INNER JOIN	yjobtitle jt
ON	emp.jobtitleid = jt.jobtitleid
WHERE	payrate <= 
		(SELECT AVG (payrate)
		FROM	yemp empINNER
		WHERE	empinner.jobtitleid = emp.jobtitleid)
ORDER BY emp.empid
```

The results from the Third Entry are shown in Fig. 4 below.

![image](BAN-702-FIG-004.jpg)

## FOURTH ENTRY
The below SQL code creates a view that calculates the average payrate by jobtitleID:

```
CREATE VIEW vAvgRateByTitle AS
SELECT 		jobtitleID,
		AVG(payrate) AveragePayRate
FROM 		yemp
GROUP BY jobtitleID;
```

By executing the below SQL code, the yEMP table will join with the newly created table.

```
SELECT 		emp.empid,
		emp.empname,
		emp.payrate,
		emp.jobtitleid,
		vAvgRateByTitle.AveragepayRate
FROM 		yemp emp
INNER JOIN 	vAvgRateByTitle
ON emp.jobtitleid = vAvgRateByTitle.jobtitleid
ORDER BY 	emp.empid
```

Add the WHERE clause to the previous query code to show only those employees who have a payrate that is greater than the AveragePayRate.

```
SELECT emp.empid,
	emp.empname,
	emp.payrate,
	emp.jobtitleid,
	vAvgRateByTitle.AveragepayRate
FROM 	yEmp emp
INNER JOIN vAvgRateByTitle
ON 	emp.jobtitleid = vAvgRateByTitle.jobtitleid
WHERE 	payrate > averagepayrate
ORDER BY emp.empid
```

The following SQL Code will only display those employees that have a payrate that is less or equal to the AveragePayRate calculated in the new view.

```
SELECT	emp.empid,
		emp.empname,
		emp.payrate,
		emp.jobtitleid,
		vAvgRateByTitle.AveragepayRate
FROM	yemp	emp
INNER JOIN	vAvgRateByTitle
ON	emp.jobtitleid = vAvgRateByTitle.jobtitleid
WHERE	payrate <= AveragePayRate
ORDER BY emp.empid
```

The results from the Fourth Entry are shown in Fig. 5 below.

![image](BAN-702-FIG-005.jpg)

## FIFTH ENTRY
The following SQL query can count orders in the yORD table by custID using the COUNT group function.

```
SELECT custID,
	count(*) CountOfOrders
FROM yord
GROUP BY custID
```

To be able to determine which customer placed the most orders the following SQL code will use the new CTE created to determine the MAX orders by customer:

```
WITH cteCountOrders AS
(SELECT		custID,
			count(*) CountOfOrders
FROM		yOrd
GROUP BY	custID)

SELECT		cust.CustID,
			CustomerName,
			CountOfOrders 
FROM		ycust cust
INNER JOIN	cteCountOrders cteCount
ON	cust.custid = ctecount.custID
WHERE	countoforders = 
		(SELECT MAX(CountOfOrders)
		FROM	cteCountOrders);
```

The results from the Fifth Entry are shown in Fig. 6 below.

![image](BAN-702-FIG-006.jpg)

## SIXTH ENTRY

Which customer(s) placed the most orders in June of the current year? The questions can be answered by using a SQL view. The following SQL Code will be able to answer this question.

```
CREATE VIEW
vJuneCount AS
SELECT	Cust.CustID,
		CustomerName,
		COUNT(*) CountofOrderLines
FROM	yCust	cust
INNER JOIN	yOrd
ON	cust.custid = yord.CustID
WHERE month(orderdate) = 6 and year (orderdate) = year(getdate())
GROUP BY	cust.CustID,  CustomerName
```

The results from the Sixth Entry are shown in Fig. 7 below.

![image](BAN-702-FIG-007.jpg)


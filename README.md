

# -Employee-Analysis-Project


## Overview
This project aims to analyze employee and salary data using advanced SQL queries and database concepts. It includes tasks involving Common Table Expressions (CTEs), stored procedures, views, ranking, pivoting, and categorization logic to gain insights from employee records, departmental structures, and salary trends.


## Data Structure
The project uses the following three primary tables:

### Employees

- Contains employee information including name, department, and hire date.

- Data to be inserted using a provided CSV file.

### Salaries

- Tracks salary history for each employee with effective dates.

- Data to be inserted using a provided CSV file.

### Departments

- Created and populated using SQL queries:

## Tools & Technologies
- SQL
- MS SQL Server
- CSV files for data insertion
- Views, CTEs, Pivoting, Ranking, Stored Procedures

## Goals
- Improve SQL querying and data analysis skills
- Learn use of CTEs, views, stored procedures, and analytical functions
- Build a portfolio-ready SQL project with real-world business queries

## Schema

### CREATED EMPLOYEES table and bulk inserted the data from csv file
``` sql CREATE TABLE Employees(
employee_id	int,
first_name	VARCHAR(25),
last_name VARCHAR(25),
department_id int,
hire_date Date)
```
```sql BULK INSERT dbo.Employees
FROM 'C:\Users\Sahana G M\OneDrive\Desktop\SQL NA Materials\SQL NA PROJECT\SQL Project-2 New\Employees.csv'
With 
(
 FORMAT='CSV',
 FIRSTROW=2
 )
```

### CREATED Salaries table and bulk inserted the data from csv file

CREATE TABLE Salaries
(employee_id int,
salary	float,
effective_date date
)

BULK INSERT dbo.salaries
FROM 'C:\Users\Sahana G M\OneDrive\Desktop\SQL NA Materials\SQL NA PROJECT\SQL Project-2 New\salaries.csv'
With
(
  FORMAT='csv',
  FIRSTROW=2
)


### Create Table Departments and insert the records using given query.
``` sql CREATE TABLE Departments( 
Department_ID INT Primary Key, 
Department_Name Varchar(20))
```

``` sql INSERT INTO Departments 
SELECT 1, 'IT' 
UNION SELECT 2,'Sales' 
UNION SELECT 3,'Finance'
```

--Q1: Retrieve employees who earn more than the average salary of their department. 
--Tables: employees, salaries 
```sql
select E.employee_id, E.first_name name, E.department_id, S.salary
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id
Where Salary>(
       SELECT AVG(s.salary)
       FROM Employees E1
       JOIN Salaries S
       ON E1.employee_id=S.employee_id
	   Where E1.department_id=E.department_id
)


```

```sql
SELECT E.employee_id,E.first_name,E.department_id,D.Department_Name
FROM Employees E
JOIN Departments D
ON E.department_id=D.Department_ID
Where E.department_id IN
(
SELECT department_id
FROM Employees 
Group By department_id
Having COUNT(employee_id)>10
)
```
--Q3: Use a CTE to find the department name with the highest total salary. 
--Tables: employees, salaries and departments

```sql
With CTE_dep as
(
SELECT sum(salary) as total_salary,E.department_id
FROM Salaries S
JOIN Employees E
ON S.employee_id=E.employee_id
Group By department_id
)
Select Department_Name
FROM CTE_dep C
Join Departments D1
ON C.department_id=D1.Department_ID
Where c.total_salary=(select MAX(total_salary)  FROM CTE_dep )

```

--Q4: Retrieve the total salary paid to employees in each department using a CTE and round the Total 
--Salary up to 2 digit afer decimal. 
--Tables: employees, departments and salaries

```sql
With CTE_Salary as
(
SELECT Round(SUM(salary),2) as total_salary,department_id
FROM Salaries S
Join Employees E
ON E.employee_id=S.employee_id
GROUP BY department_id
)
Select 
    CS.department_id,
	D.Department_Name,
    CS.total_salary
FROM CTE_Salary CS
JOIN Departments D
On D.Department_ID=CS.department_id
```
--Q5: Create a stored procedure to retrieve all the employees from a given department. 
--Tables: employees 

```sql
CREATE PROCEDURE SP_Employees
@depaartmentId int
AS
BEGIN
        IF @depaartmentId IS NULL
          Begin
            Print('Department is not Present')
	       Return
	      End

SELECT * FROM Employees E
Where @depaartmentId= E.department_id
END

EXEC SP_Employees @departmentId='102'
```
--Q6: Create a stored procedure to get the total salary of a department. 
--Tables: employees, salaries

```sql
DROP Procedure Totalsalary
CREATE PROCEDURE Totalsalary
@dept_id int
As
BEGIN
(
     SELECT SUM(salary),E.department_id
     FROM Employees E
	 JOIN Salaries S
	 ON E.employee_id=S.employee_id
	 --Where department_id=@dept_id
	 GROUP BY E.department_id
	
	 )
END

EXEC Totalsalary @dept_id = 101

```

--Q7: Create a view to show the employee name and their current salary. 
--Tables: employees, salaries

```sql
CREATE VIEW V_Employee
As
(
SELECT E.first_name as name,S.salary
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id
)

SELECT * FROM V_Employee

```


--Q8: Count employees by department, distinguishing between those hired before and afer year 2010. 
--Tables: employees, departments 
```sql
WITH CTE_Year_comparision AS 
(
         SELECT department_id,
		        SUM(case When hire_date<'2010-01-01' Then 1 ELSE 0 END) as hired_before2010,
				SUM(case when hire_date>='2010-01-01' THEN 1 ELSE 0 END) as hired_after2010
         FROM Employees 
         GROUP BY department_id	
)
Select D.Department_Name,
       C.hired_before2010,
	   C.hired_after2010
FROM CTE_Year_comparision C
JOIN Departments D
ON D.Department_ID=C.Department_ID
```


--Q9: Categorize employees based on their salary ranges. IF Salary< 30000 Then "Low",  
--IF Greater than equal to 30000 and Less then 60000 then "Medium" Else "High" 
--Tables: employees, salaries 

```sql
SELECT E.first_name as name,
           E.department_id,
		   S.salary,
           New_salary=
                      CASE
					  When salary<30000 then 'Low'
					  When (salary>=30000 and salary<60000) then 'Medium'
					  Else 'high'
					  END

FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id

```

--Q10: Retrieve the previous salary of each employee. 
--Tables: employees, salaries 

```sql
SELECT *,
LAG(salary) OVER( PARTITION BY E.employee_id order by effective_date)
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id
```

--Q11: Retrieve the first salary of each employee based on starting Effective_date. 
--Tables: employees, salaries

```sql
SELECT E.employee_id,
       E.first_name name,
	   S.salary as current_salary,
	   S.effective_date as date,
      FIRST_VALUE(S.salary) OVER(PARTITION BY E.employee_id Order BY S.effective_date) as first_Salary
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id

```
--Q12: Retrieve the most recent salary of each employee. (Take Effective_date to find the salary 
--change date)  
--Tables: employees, salaries 

```sql
WITH Recent_salary AS
(
SELECT E.employee_id,
       E.first_name As name,
	   S.salary as Current_salary,
      ROW_number()  OVER(Partition BY e.employee_id Order by S.effective_date) as R_N
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id
)
SELECT employee_id,
       name,
	   Current_salary
FROM Recent_salary
Where R_N=1

```
--Q13: Write a query to rank departments based on their total salaries and count the number of 
--employees in each department. 
--Tables: employees, departments, salaries 

```sql
WITH CTE_RankEMployee AS
(
SELECT 
      SUM(S.salary) total_salary,
	  COUNT(E.employee_id) Count_of_employee,
	  D.department_name
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id
JOIN Departments D
ON E.department_id=D.Department_ID
GROUP BY D.Department_Name
)
SELECT CR.department_name,
       CR.total_salary,
       CR.Count_of_employee,
       RANK() OVER (Order BY total_salary DESC) as Dept_Rank
FROM CTE_RankEMployee CR

```
--Q14: Using Mul-Level CTE Calculate the average salary for each department and rank departments 
--based on the average salary. 
--Tables: employees, departments, salaries

```sql
WITH Average_salary AS
(
SELECT AVG(S.salary) as avg_salary,
       D.Department_Name
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id
JOIN Departments D
ON E.department_id=D.Department_ID
GROUP BY D.Department_Name
)
SELECT A.department_name,
       A.avg_salary,
       RANK() OVER(Order BY avg_salary DESC) as Dept_avg_salary_rank
FROM Average_salary A
```
--Q15: Write a query to calculate the cumulative sum of salaries for each department ordered by 
--employee hire date. 
--Tables: employees, departments, salaries 

```sql
SELECT E.employee_id,
       E.first_name,
       D.Department_Name,
       S.salary,
       E.hire_date,
	   CUME_DIST() OVER (PARTITION BY D.Department_Name
            ORDER BY E.hire_date)
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id
JOIN Departments D
ON E.department_id=D.Department_ID
```
--Q16: Use a nested CTE to find the department with the most employees. 
--Tables: employees, departments
```sql
WITH CTE_Dept AS
(
       SELECT COUNT(E.employee_id) num_Employee,D.Department_Name
       FROM Employees E
       JOIN Departments D
       ON E.department_id=D.Department_ID
       GROUP BY D.Department_Name
),
  CTE_MaxEmployee AS
   (
   SELECT Top 1*
   FROM CTE_Dept
   Order By num_Employee DESC
   )
SELECT * 
FROM CTE_MaxEmployee

```
--Q17: Find total salary paid per year for each department using Pivot Table. 
--Tables: employees, departments, salaries

```sql
WITH Salary_Year_Department AS (
    SELECT 
        YEAR(S.effective_date) AS Salary_Year,
        D.department_name,
        S.salary
    FROM Employees E
    JOIN Salaries S ON E.employee_id = S.employee_id
    JOIN Departments D ON E.department_id = D.Department_ID
)


SELECT *
FROM Salary_Year_Department
PIVOT(
       sum(salary)
	   FOR department_name IN([Finanace],[IT],[Sales])) as P1

```
--Q18: Write a query to calculate the cumulative sum of salaries for each department ordered by 
--employee hire date. 
--Tables: employees, salaries 

```sql
SELECT E.department_id,
       E.employee_id,
	   E.hire_date,
	   S.salary,
	   SUM(S.salary) OVER(Partition BY E.department_id ORDER BY E.hire_date) as Cum_sum_salary
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id

```
--Q19: Write a query to calculate a rolling average salary over the last 3 salary entries for each 
--employee 
--Tables: employees, salaries 

```sql

With CTE_Rolling_avg_salary As
(
SELECT E.employee_id,
       E.department_id,
	   S.salary,
	   AVG(S.salary) OVER(PARTITION BY E.employee_id ORDER BY E.hire_date DESC) as avg_salary
FROM Employees E
JOIN Salaries S
ON E.employee_id=S.employee_id
)
SELECT TOP 3*
FROM CTE_Rolling_avg_salary C

```
--Q20: Write a query to analyse employee  retention  by calculating the percentage of employees 
--remaining at the end of each year. 
--Tables: employees 

```sql
SELECT *,
     PERCENT_RANK() OVER(Partition by hire_date ORDER BY employee_id) as Retention_of_employee
FROM Employees E

WITH HireStats AS (
    SELECT 
        YEAR(hire_date) AS hire_year,
        COUNT(*) AS hired_i
		n_year,
        SUM(COUNT(*)) OVER (ORDER BY YEAR(hire_date)) AS cumulative_hired
    FROM Employees
    GROUP BY YEAR(hire_date)
)
SELECT 
    hire_year,
    hired_in_year,
    cumulative_hired,
	E.first_name,
    ROUND((cumulative_hired * 100.0) / SUM(hired_in_year) OVER (), 2) AS retention_rate_percentage
FROM HireStats H
JOIN Employees E
ON E.hire_year=H.hire_year
ORDER BY hire_year
```



  ## Findings and Conclusion

This Employee Analysis project provides a comprehensive exploration of employee, salary, and departmental data using advanced SQL techniques. By working through a range of realistic business queries—including salary trends, departmental insights, employee retention, and role-based data retrieval—this project strengthens core data analysis and database management skills.

It demonstrates the effective use of:
Common Table Expressions (CTEs) for modular and readable queries,
Stored procedures for reusable operations,
Views for abstraction and simplified data access,
Analytical functions like ranking, cumulative sums, and rolling averages,
Pivoting for multidimensional analysis.
Such practices are critical in real-world data analytics roles, especially in HR analytics, payroll management, and strategic workforce planning.

Now SQL Practice 🔥

Use these tables:

Employee
Eid	Name	DeptId	Salary	City
1	Amit	10	50000	Delhi
2	Neha	20	70000	Mumbai
3	Ravi	10	80000	Delhi
4	Pooja	30	60000	Jaipur
5	Karan	10	NULL	Mumbai
6	Rahul	20	90000	Delhi


Department
DeptId	DeptName
10	IT
20	HR
30	Finance
40	Legal

Try these yourself.

Display all employees. -> select * from employee.
Display only employee Name and Salary. -> Select name, salary from employee.
Find employees who belong to Delhi. -> select * from employee where city='Delhi'
Find employees whose salary is greater than 60000. -> select * from employee where Salary > 6000;
Find employees whose salary is between 50000 and 80000. -> select * from employee where Salary between 5000 and 60000;
Find employees whose name starts with R. -> Select * from employee where name = 'R%'
Find employees whose second character is a. ->   Select * from employee where name = '_a%'
Find employees whose salary is NULL. ->  Select * from employee where Salary is NULL;
Display all unique cities. -> Select UNIQUE(City) from Employee;
Sort employees by salary in descending order. -> select * from employee ORDER BY Salary Desc;
Find total number of employees. -> Select Count(*) from employee;
Find number of employees whose salary is not NULL. -> Select Count(Salary) from employee;
Find highest salary. -> Select Max(Salary) from employee;
Find lowest salary. Select Min(Salary) from employee;
Find average salary. -> Select Avg(Salary) from employee;
Find department-wise employee count. -> Select Count(DeptId) from employee GROUP BY DeptId
Find department-wise average salary. -> Select DeptId, Avg(Salary) from employee GROUP BY DeptId
Find departments having more than 2 employees. -> Select Count(DeptId) from employee GROUP BY DeptId HAVING count(deptId) > 2;
Find department-wise maximum salary. -> Select Max(Salary), deptId from employee GROUP BY DeptId;
Find only those departments whose average salary is greater than 65000. Select DeptName,DeptId from employee.DeptId = Department.DeptId 

Slightly harder
Find number of distinct cities in which employees live.
Find department-wise number of employees from Delhi.
Find departments where at least 2 employees have salary greater than 50000.
Find the total salary paid to employees of each department.
Display departments in descending order of their average salary.

And one conceptual challenge:


                                                    JOINS IN SQL. -> A join in SQL is used to combine related rows from two or more tables.


Employee

| EmpId | Name  | DeptId |
| ----: | ----- | -----: |
|     1 | Amit  |     10 |
|     2 | Neha  |     20 |
|     3 | Ravi  |     10 |
|     4 | Pooja |     30 |
|     5 | Karan |     50 |


Department.

| DeptId | DeptName |
| -----: | -------- |
|     10 | IT       |
|     20 | HR       |
|     30 | Finance  |
|     40 | Legal    |


Employee.DeptId is the foreign key.
Department.DeptId is the referenced key.


1. Inner Join -> An inner join returns only those rows for which a matching row is present in both the tables.
SQL Query -> Select  E.Name, D.DeptName From Employee E INNER JOIN Department D ON E.DeptId = D.DeptId
               Amit IT
               Neha HR
               Ravi Fianance

ON clause -> Defines the matching row conditions. 

2. Equi Join -> A join in which the comparision operator is =
SELECT *
FROM Employee E
JOIN Department D
ON E.DeptId = D.DeptId;

3. Thetha Join -> A join which allows comparison operators such as :

=
>
<
>=
<=
<>


`4. Natural Join` -> A natural join automatically looks for columns having same name in both the tables.
A natural join normally keeps the common join column only once in the output.

SELECT * FROM Employee NATUAL JOIN Department.

Conceptually -> Employee.DeptId = Department.DeptId.

`5. Outer Join` -> Inner join removes unmatched rows, an outer join may preserver unmatched rows. There are three different outer joins.
                a. LEFT OUTER JOIN
                b. RIGHT OUTER JOIN
                c. FULL OUTER JOIN.

a. LEFT OUTER JOIN / LEFT JOIN -> All rows from the left table, plus matching rows from right table.

Query : Select * from Employee E LEFT JOIN Deppartment D ON E.DeptId = D.DeptId.
Output : 
| Name  | DeptId | DeptName |
| ----- | -----: | -------- |
| Amit  |     10 | IT       |
| Neha  |     20 | HR       |
| Ravi  |     10 | IT       |
| Pooja |     30 | Finance  |
| Karan |     50 | NULL     |


b. RIGHT OUTER JOIN/ RIGHT JOIN -> It is just the opposite of Left join. It returns all rows from right tables,plus matching rows from left table.
SQL : Select E.Name, D.DeptId, D.DeptName  from Employee E RIGHT JOIN Department D ON E.DeptId = D.DeptId

| Name  | DeptId | DeptName |
| ----- | -----: | -------- |
| Amit  |     10 | IT       |
| Ravi  |     10 | IT       |
| Neha  |     20 | HR       |
| Pooja |     30 | Finance  |
| NULL  |     40 | Legal    |



c. FULL OUTER JOIN -> Matching rows are combined and unmatched rows from both the tables are also preserved.


`6. SELF JOIN` -> A self join means:
A table is joined with itself.
Suppose:

Employee
EmpId	Name	ManagerId
1	Amit	NULL
2	Neha	1
3	Ravi	1
4	Pooja	2

Here:

ManagerId

also refers to an employee.

For example:

Neha.ManagerId = 1

and:

EmpId 1 = Amit

Therefore Amit is Neha's manager.

To find employee and manager names:

SELECT E.Name AS Employee,
       M.Name AS Manager
FROM Employee E
LEFT JOIN Employee M
ON E.ManagerId = M.EmpId;

Here the same table is being treated as two logical copies:

E = Employee
M = Manager
Important

Self join is not a completely separate join algorithm.

It means:

Joining a relation with itself using aliases.

`7. CROSS JOIN` -> A cross join creates the cartesian product. 
SQL -> Select * From Employee CROSS JOIN Department. (Result = 5 rows * 4 rows = 20 rows)



15. SQL Join Summary
Join	Result
INNER JOIN	Only matching rows
LEFT JOIN	All left + matching right
RIGHT JOIN	All right + matching left
FULL OUTER JOIN	All rows from both
CROSS JOIN	Every possible combination
SELF JOIN	Table joined with itself
NATURAL JOIN	Automatically joins same-named common attributes
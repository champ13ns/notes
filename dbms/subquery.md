``SUBQUERY`` -> A subquery is a query written inside a query.

Suppose we have this table:

Employee

EmpId	Name	DeptId	Salary
1	Amit	10	50000
2	Neha	20	70000
3	Ravi	10	80000
4	Pooja	30	60000
5	Rahul	20	90000

Question:
Find employees whose salary is greater than the average salary.

Select * from Employee where (
    Select AVG(Salary) from Employee
) > 70000

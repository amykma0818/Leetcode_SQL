### Leetcode 1978: Employees Whose Manager Left the Company
Write an SQL query to report the IDs of the employees whose salary is strictly less than $30000 and whose manager left the company. When a manager leaves the company, their information is deleted from the Employees table, but the reports still have their manager_id set to the manager that left.

Return the result table ordered by employee_id.
``` Mysql
select employee_id 
from Employees
where salary<30000 and manager_id not in (select employee_id from Employees) 
order by employee_id
```

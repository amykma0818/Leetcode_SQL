### Leetcode 1990. Count the Number of Experiments (Medium)

experiment_id is the primary key for this table.
platform is an enum with one of the values ('Android', 'IOS', 'Web').
experiment_name is an enum with one of the values ('Reading', 'Sports', 'Programming').
This table contains information about the ID of an experiment done with a random person, the platform used to do the experiment, and the name of the experiment.

Write an SQL query to report the number of experiments done on each of the three platforms for each of the three given experiments. Notice that all the pairs of (platform, experiment) should be included in the output including the pairs with zero experiments.

Return the result table in any order.

``` MySQL
with cte1 as (select 'Android' as platform
union
select'IOS' as platform
union
select 'Web' as platform ),
cte2 as (select 'Reading' as experiment_name
union
select'Sports' as experiment_name
union
select 'Programming' asexperiment_name ),
cte3 as (select * from cte1 cross join cte2)

select c.platform, c.experiment_name, ifnull(count(experiment_id),0) num_experiments
from cte3 c left join Experiments a
on c.platform=a.platform and c.experiment_name= a.experiment_name
group by 1,2
```

### Leetcode 1978: Employees Whose Manager Left the Company (Easy)
Write an SQL query to report the IDs of the employees whose salary is strictly less than $30000 and whose manager left the company. When a manager leaves the company, their information is deleted from the Employees table, but the reports still have their manager_id set to the manager that left.

Return the result table ordered by employee_id.
``` Mysql
select employee_id 
from Employees
where salary<30000 and manager_id not in (select employee_id from Employees) 
order by employee_id
```

### Leetcode 2004. The Number of Seniors and Juniors to Join the Company

A company wants to hire new employees. The budget of the company for the salaries is $70000. The company's criteria for hiring are:

Hiring the largest number of seniors.
After hiring the maximum number of seniors, use the remaining budget to hire the largest number of juniors.
Write an SQL query to find the number of seniors and juniors hired under the mentioned criteria.

Return the result table in any order.

``` Mysql
with cte as (
    select employee_id, experience, 
    sum(salary) over(partition by experience order by salary asc) as cum_salary
    from Candidates
)

select 'Senior' as experience, count(*) as accepted_candidates
from cte where cum_salary<=70000 and experience="Senior"
union all
select 'Junior' as experience, count(*) as accepted_candidates
from cte where experience="Junior" and cum_salary<= 
70000 - (select ifnull(max(cum_salary),0) from cte where experience="Senior" and cum_salary<70000 )
```
### Leetcode 1972. First and Last Call On the Same Day
Write an SQL query to report the IDs of the users whose first and last calls on any day were with the same person. Calls are counted regardless of being the caller or the recipient.

Return the result table in any order.

``` Mysql
with cte as (
select * from Calls
    union all
select recipient_id as caller_id,
    caller_id as recipient_id,
    call_time from Calls
)

select distinct caller_id as user_id from 
(select caller_id, DATE(call_time) as day, 
first_value(recipient_id) over(partition by caller_id, DATE(call_time) order by call_time asc) as first_call,
first_value(recipient_id) over(partition by caller_id, DATE(call_time) order by call_time desc) as last_call
from cte) t
where first_call=last_call
```
### Leetcode 1965. Employees With Missing Information
Write an SQL query to report the IDs of all the employees with missing information. The information of an employee is missing if:

The employee's name is missing, or
The employee's salary is missing.
Return the result table ordered by employee_id in ascending order.

``` Mysql
select employee_id from Employees
where employee_id not in (select employee_id from Salaries)
union 
select employee_id from Salaries
where employee_id not in (select employee_id from Employees)
order by employee_id asc
```
### Leetcode 1951. All the Pairs With the Maximum Number of Common Followers

Write an SQL query to find all the pairs of users with the maximum number of common followers. In other words, if the maximum number of common followers between any two users is maxCommon, then you have to return all pairs of users that have maxCommon common followers.

The result table should contain the pairs user1_id and user2_id where user1_id < user2_id.

Return the result table in any order.

``` Mysql
with cte as (
select a.user_id as user1_id,
    b.user_id as user2_id, count(distinct a.follower_id) as common
from Relations a, Relations b
where a.follower_id=b.follower_id and a.user_id<b.user_id
group by a.user_id, b.user_id
)

select user1_id, user2_id 
from cte where common=(select max(common) from cte)
```

### Leetcode 1949. Strong Friendship
A friendship between a pair of friends x and y is strong if x and y have at least three common friends. Note x and y have to be friends.

Write an SQL query to find all the strong friendships.

Note that the result table should not contain duplicates with user1_id < user2_id. 

Return the result table in any order.

``` Mysql
with cte as (
select * from Friendship
    union
select user2_id as user1_id,
    user1_id as user2_id from Friendship
)


select a.user1_id as user1_id, b.user1_id as user2_id,
count(distinct a.user2_id) as common_friend
from cte a, cte b
where a.user1_id<b.user1_id and a.user2_id=b.user2_id 
and (a.user1_id, b.user1_id) in (select * from cte)
group by user1_id, user2_id
having common_friend>=3
```

### Leetcode 1939. Users That Actively Request Confirmation Messages
Write an SQL query to find the IDs of the users that requested a confirmation message twice within a 24-hour window. Two messages exactly 24 hours apart are considered to be within the window. The action does not affect the answer, only the request time.

Return the result table in any order.
``` mysql
select distinct a.user_id 
from Confirmations a, Confirmations b
where a.user_id=b.user_id and a.time_stamp<b.time_stamp
and b.time_stamp <= DATE_ADD(a.time_stamp, Interval 24 HOUR) 
```






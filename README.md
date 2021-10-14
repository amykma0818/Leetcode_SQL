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

### Leetcode 1934. Confirmation Rate
The confirmation rate of a user is the number of 'confirmed' messages divided by the total number of requested confirmation messages. The confirmation rate of a user that did not request any confirmation messages is 0. Round the confirmation rate to two decimal places.

Write an SQL query to find the confirmation rate of each user.

Return the result table in any order.

``` mysql
with cte as(
select distinct user_id, 
sum(case when action="confirmed" then 1 else 0 end ) over(partition by user_id)/count(action) over(partition by user_id) as rate
from Confirmations
)

select a.user_id, ifnull(round(b.rate,2),0) as confirmation_rate
from Signups as a 
left join cte as b
on a.user_id=b.user_id
```

### Leetcode 1988. Find Cutoff Score for Each School

### Leetcode 1919. Leetcodify Similar Friends
Write an SQL query to report the similar friends of Leetcodify users. A user x and user y are similar friends if:

Users x and y are friends, and
Users x and y listened to the same three or more different songs on the same day.
Return the result table in any order. Note that you must return the similar pairs of friends the same way they were represented in the input (i.e., always user1_id < user2_id).

``` mysql
with cte as (
select a.user_id as user1_id, b.user_id as user2_id,
    count(distinct a.song_id) as common
    from Listens a, Listens b
    where a.user_id<b.user_id and a.song_id=b.song_id and a.day=b.day
    group by a.user_id, b.user_id, a.day
    having common>=3
)

select distinct user1_id, user2_id
from cte where (user1_id,user2_id) in (select * from Friendship)
```
### Leetcode 1917. Leetcodify Friends Recommendations
Write an SQL query to recommend friends to Leetcodify users. We recommend user x to user y if:

Users x and y are not friends, and
Users x and y listened to the same three or more different songs on the same day.
Note that friend recommendations are unidirectional, meaning if user x and user y should be recommended to each other, the result table should have both user x recommended to user y and user y recommended to user x. Also, note that the result table should not contain duplicates (i.e., user y should not be recommended to user x multiple times.).

Return the result table in any order.

``` mysql
with cte as (
select * from Friendship
    union
select user2_id as user1_id, user1_id as user2_id
    from Friendship
)

select distinct a.user_id, b.user_id as recommended_id 
    from Listens as a 
    join Listens as b
    on a.day=b.day and a.user_id!=b.user_id and a.song_id=b.song_id
    where (a.user_id, b.user_id) not in (select * from cte)
    group by a.user_id, b.user_id,a.day
    having count(distinct a.song_id)>=3
```
### Leetcode 2010. The Number of Seniors and Juniors to Join the Company II
A company wants to hire new employees. The budget of the company for the salaries is $70000. The company's criteria for hiring are:

Keep hiring the senior with the smallest salary until you cannot hire any more seniors.
Use the remaining budget to hire the junior with the smallest salary.
Keep hiring the junior with the smallest salary until you cannot hire any more juniors.

Write an SQL query to find the ids of seniors and juniors hired under the mentioned criteria. Return the result table in any order.

``` mysql
with cte as (
select * from 
(select employee_id, experience, 
    sum(salary) over(partition by experience order by salary asc) as cum_salary
    from Candidates) t where cum_salary<=70000
)

select employee_id from cte where experience="Senior"
union
select employee_id from cte where experience="Junior" and 
cum_salary<= 70000 - (select ifnull(max(cum_salary),0) from cte where experience="Senior" )
```

### Leetcode 1907. Count Salary Categories
Write an SQL query to report the number of bank accounts of each salary category. The salary categories are:

* "Low Salary": All the salaries strictly less than $20000.
* "Average Salary": All the salaries in the inclusive range [$20000, $50000].
* "High Salary": All the salaries strictly greater than $50000.

The result table must contain all three categories. If there are no accounts in a category, then report 0. Return the result table in any order.

The query result format is in the following example.

``` mysql
select "Low Salary" as category, count(distinct account_id) as accounts_count 
from Accounts where income<20000
union all
select "Average Salary" as category, count(distinct account_id) as accounts_count 
from Accounts where income>=20000 and income<=50000
union all
select "High Salary" as category, count(distinct account_id) as accounts_count 
from Accounts where income>50000
```
### Leetcode 1892. Page Recommendations II
You are implementing a page recommendation system for a social media website. Your system will recommended a page to user_id if the page is liked by at least one friend of user_id and is not liked by user_id.

Write an SQL query to find all the possible page recommendations for every user. Each recommendation should appear as a row in the result table with these columns:

* user_id: The ID of the user that your system is making the recommendation to.
* page_id: The ID of the page that will be recommended to user_id.
* friends_likes: The number of the friends of user_id that like page_id.

Return result table in any order.
``` mysql 
with cte as (
select * from Friendship
    union
select user2_id as user1_id, user1_id as user2_id from Friendship
),
cte1 as(
select a.user1_id, b.page_id, count(distinct a.user2_id) as friends_likes
from cte as a 
left join Likes as b
on a.user2_id=b.user_id
group by a.user1_id, b.page_id
)

select c.user1_id as user_id, c.page_id,c.friends_likes
from cte1 c
left join Likes d
on c.user1_id=d.user_id and c.page_id=d.page_id
where d.page_id is null
```
### Leetcode 1890. The Latest Login in 2020
Write an SQL query to report the latest login for all users in the year 2020. Do not include the users who did not login in 2020.

Return the result table in any order.
``` mysql
select user_id, max(time_stamp) as last_stamp
from Logins where year(time_stamp)=2020
group by user_id
```
### Leetcode 1875. Group Employees of the Same Salary
A company wants to divide the employees into teams such that all the members on each team have the same salary. The teams should follow these criteria:

Each team should consist of at least two employees.
All the employees on a team should have the same salary.
All the employees of the same salary should be assigned to the same team.
If the salary of an employee is unique, we do not assign this employee to any team.
A team's ID is assigned based on the rank of the team's salary relative to the other teams' salaries, where the team with the lowest salary has team_id = 1. Note that the salaries for employees not on a team are not included in this ranking.
Write an SQL query to get the team_id of each employee that is in a team.

Return the result table ordered by team_id in ascending order. In case of a tie, order it by employee_id in ascending order.
``` mysql
with cte as (
select *, count(employee_id) over(partition by salary) as counts
    from Employees
)

select employee_id, name, salary, dense_rank() over(order by salary asc) as team_id
from cte where counts>1
order by team_id asc, employee_id asc
```
### Leetcode 1873. Calculate Special Bonus
Write an SQL query to calculate the bonus of each employee. The bonus of an employee is 100% of their salary if the ID of the employee is an odd number and the employee name does not start with the character 'M'. The bonus of an employee is 0 otherwise.

Return the result table ordered by employee_id.
``` mysql
select employee_id, 
(case when employee_id%2!=0 and left(name,1)!="M" then salary
else 0 end) as bonus
from Employees
order by employee_id
```
### Leetcode 1867. Orders With Maximum Quantity Above Average
You are running an ecommerce site that is looking for imbalanced orders. An imbalanced order is one whose maximum quantity is strictly greater than the average quantity of every order (including itself).

The average quantity of an order is calculated as (total quantity of all products in the order) / (number of different products in the order). The maximum quantity of an order is the highest quantity of any single product in the order.

Write an SQL query to find the order_id of all imbalanced orders.

Return the result table in any order.
``` mysql
with cte as 
(select *, sum(quantity) over(partition by order_id)/count(product_id) over(partition by order_id) as avg_quan, max(quantity) over(partition by order_id) as max_quan
from OrdersDetails) 

select distinct order_id from cte
where max_quan> (select max(avg_quan) from cte)
```
### Leetcode 1853. Convert Date Format
Write an SQL query to convert each date in Days into a string formatted as "day_name, month_name day, year".

Return the result table in any order.
``` mysql
select concat(date_format(day,"%W"),", ",date_format(day,"%M %e"),", ",date_format(day,"%Y")) as day
from Days
```
### Leetcode 1843. Suspicious Bank Accounts
Write an SQL query to report the IDs of all suspicious bank accounts.

A bank account is suspicious if the total income exceeds the max_income for this account for two or more consecutive months. The total income of an account in some month is the sum of all its deposits in that month (i.e., transactions of the type 'Creditor').

Return the result table in ascending order by transaction_id.
``` mysql
with cte as (
select a.account_id, sum(a.amount) as sum_amount, b.max_income,
  date_format(a.day,"%Y%m") as month
from Transactions as a
join Accounts as b
on a.account_id=b.account_id
where a.type="Creditor"
group by a.account_id, date_format(a.day,"%Y%m")
having sum_amount>max_income
)

select distinct a.account_id
from cte a, cte b
where a.account_id=b.account_id and period_diff(a.month, b.month)=-1
order by account_id
```
### Leetcode 1841. League Statistics
Write an SQL query to report the statistics of the league. The statistics should be built using the played matches where the winning team gets three points and the losing team gets no points. If a match ends with a draw, both teams get one point.

Each row of the result table should contain:

team_name - The name of the team in the Teams table.
matches_played - The number of matches played as either a home or away team.
points - The total points the team has so far.
goal_for - The total number of goals scored by the team across all matches.
goal_against - The total number of goals scored by opponent teams against this team across all matches.
goal_diff - The result of goal_for - goal_against.

Return the result table in descending order by points. If two or more teams have the same points, order them in descending order by goal_diff. If there is still a tie, order them by team_name in lexicographical order.

``` mysql
with cte as (
select home_team_id as team_id, home_team_goals as goals_for,
    away_team_goals as goals_against,
    (case when home_team_goals>away_team_goals then 3
    when home_team_goals=away_team_goals then 1
    else 0 end) as points
from Matches
    union all
select away_team_id as team_id, away_team_goals as goals_for,
    home_team_goals as goals_against,
    (case when home_team_goals<away_team_goals then 3
    when home_team_goals=away_team_goals then 1
    else 0 end) as points
from Matches
)

select b.team_name, count(a.team_id) as matches_played, sum(a.points) as points,
sum(a.goals_for) as goal_for, sum(a.goals_against) as goal_against,
sum(a.goals_for)-sum(a.goals_against) as goal_diff
from cte a
join Teams b
on a.team_id=b.team_id
group by b.team_name
order by points desc, goal_diff desc, team_name asc
```

### Leetcode 1831. Maximum Transaction Each Day
Write an SQL query to report the IDs of the transactions with the maximum amount on their respective day. If in one day there are multiple such transactions, return all of them.

Return the result table in ascending order by transaction_id.
``` mysql
select transaction_id from
(select *, rank() over(partition by date(day) order by amount desc) as rnk
from Transactions) t 
where rnk=1 order by transaction_id asc
```
### Leetcode 2020. Number of Accounts That Did Not Stream
Write an SQL query to report the number of accounts that bought a subscription in 2021 but did not have any stream session.
``` mysql
select count(account_id) as accounts_count  
from Subscriptions
where (year(start_date)=2021 or year(end_date)=2021) 
and account_id not in (select account_id from Streams where year(stream_date)=2021)
```
### Leetcode 1821. Find Customers With Positive Revenue this Year
Write an SQL query to report the customers with postive revenue in the year 2021.

Return the result table in any order.
``` mysql
select distinct customer_id from Customers
where year=2021 and revenue>0
```
### Leetcode 1811. Find Interview Candidates
Write an SQL query to report the name and the mail of all interview candidates. A user is an interview candidate if at least one of these two conditions is true:

The user won any medal in three or more consecutive contests.
The user won the gold medal in three or more different contests (not necessarily consecutive).
Return the result table in any order.
```mysql
with cte as (
select contest_id, gold_medal as medal from Contests
union all
select contest_id, silver_medal as medal from Contests
union all
select contest_id, bronze_medal as medal from Contests
),
cte1 as (
select medal 
from (select *, contest_id-row_number() over(partition by medal order by contest_id) as rnk from cte) t 
group by medal, rnk
having count(*)>=3
),
cte2 as (
select * from cte1
union
select gold_medal as medal from Contests
group by gold_medal
having count(*)>=3
)
 
select b.name, b.mail from cte2 as a
left join Users as b
on a.medal=b.user_id
```
### Leetcode 1809. Ad-Free Sessions
Write an SQL query to report all the sessions that did not get shown any ads.

Return the result table in any order.
```mysql
select session_id from Playback 
where session_id not in
(select a.session_id from Playback as a 
join Ads as b
on a.customer_id=b.customer_id
where b.timestamp between a.start_time and a.end_time) 

```
### Leetcode 1795. Rearrange Products Table
Write an SQL query to rearrange the Products table so that each row has (product_id, store, price). If a product is not available in a store, do not include a row with that product_id and store combination in the result table.

Return the result table in any order.
```mysql
select product_id, 'store1' as store, store1 as price
from Products
where store1 is not null
union 
select product_id, 'store2' as store, store2 as price
from Products
where store2 is not null
union
select product_id, 'store3' as store, store3 as price
from Products
where store3 is not null
```
### Leetcode 1789. Primary Department for Each Employee
Employees can belong to multiple departments. When the employee joins other departments, they need to decide which department is their primary department. Note that when an employee belongs to only one department, their primary column is `'N'`.

Write an SQL query to report all the employees with their primary department. For employees who belong to one department, report their only department.

Return the result table in any order.
```mysql
select employee_id, department_id from Employee
where primary_flag='Y' or
employee_id not in (select employee_id from Employee where primary_flag='Y')
```
### Leetcode 2026. Low-Quality Problems
Write an SQL query to report the IDs of the low-quality problems. A LeetCode problem is low-quality if the like percentage of the problem (number of likes divided by the total number of votes) is strictly less than 60%.

Return the result table ordered by problem_id in ascending order.
```mysql
select problem_id from Problems
where likes/(likes+dislikes)<0.6
order by problem_id asc
```
### Leetcode 1783. Grand Slam Titles
Write an SQL query to report the number of grand slam tournaments won by each player. Do not include the players who did not win any tournament.

Return the result table in any order.
```mysql
with cte as (
select id, count(*) as grand_slams_count
from (
select Wimbledon as id from Championships
    union all
select Fr_open as id from Championships
    union all
select US_open as id from Championships
     union all
select AU_open as id from Championships
) t
group by id    
)

select a.id as player_id, b.player_name, a.grand_slams_count
from cte as a 
left join Players as b
on a.id=b.player_id
```
### Leetcode 1777. Product's Price for Each Store

Write an SQL query to find the price of each product in each store.

Return the result table in any order.
```mysql
select product_id,
sum(case when store="store1" then price else null end) as "store1",
sum(case when store="store2" then price else null end) as "store2",
sum(case when store="store3" then price else null end) as "store3"
from Products
group by product_id
```
Note: sum() can't be ignored in this case.

### Leetcode 1767. Find the Subtasks That Did Not Execute
Write an SQL query to report the IDs of the missing subtasks for each task_id.

Return the result table in any order.
```mysql
with recursive cte as (
select task_id, subtasks_count from Tasks
    union all
select task_id, subtasks_count-1 
    from cte where subtasks_count>1
)

select a.task_id, a.subtasks_count as subtask_id
from cte as a
left join Executed as b
on a.task_id=b.task_id and a.subtasks_count=b.subtask_id
where b.subtask_id is null
```

### Leetcode 1757. Recyclable and Low Fat Products
Write an SQL query to find the ids of products that are both low fat and recyclable.

Return the result table in any order.
```mysql
select product_id from Products
where low_fats="Y" and recyclable="Y"
```
### Leetcode 1747. Leetflex Banned Accounts
Write an SQL query to find the account_id of the accounts that should be banned from Leetflex. An account should be banned if it was logged in at some moment from two different IP addresses.

Return the result table in any order.
```mysql
select distinct a.account_id 
from LogInfo as a
join LogInfo as b
on a.account_id=b.account_id 
where (b.login>=a.login and b.login<=a.logout) and a.ip_address!=b.ip_address
```
### 1741. Find Total Time Spent by Each Employee
Write an SQL query to calculate the total time in minutes spent by each employee on each day at the office. Note that within one day, an employee can enter and leave more than once. The time spent in the office for a single entry is out_time - in_time.

Return the result table in any order
```mysql
select event_day as day, emp_id, sum(out_time-in_time) as total_time
from Employees
group by emp_id, day
```
### 1731. The Number of Employees Which Report to Each Employee
For this problem, we will consider a manager an employee who has at least 1 other employee reporting to them.

Write an SQL query to report the ids and the names of all managers, the number of employees who report directly to them, and the average age of the reports rounded to the nearest integer.

Return the result table ordered by employee_id.
```mysql
select b.employee_id, b.name, count(a.employee_id) as reports_count, 
round(avg(a.age),0) as average_age
from Employees as a
join Employees as b
on a.reports_to=b.employee_id 
group by b.employee_id
order by employee_id
```

### 1729. Find Followers Count
Write an SQL query that will, for each user, return the number of followers.

Return the result table ordered by user_id.
```mysql
select user_id, count(*) as followers_count
from Followers 
group by user_id
order by user_id asc
```
### 1715. Count Apples and Oranges
Write an SQL query to count the number of apples and oranges in all the boxes. If a box contains a chest, you should also include the number of apples and oranges it has.

Return the result table in any order.
```mysql
select sum(a.apple_count+ifnull(b.apple_count,0)) as apple_count,
sum(a.orange_count+ifnull(b.orange_count,0)) as orange_count
from Boxes as a
left join Chests as b
on a.chest_id=b.chest_id
```

### Leetcode 1709. Biggest Window Between Visits
Assume today's date is '2021-1-1'.

Write an SQL query that will, for each user_id, find out the largest window of days between each visit and the one right after it (or today if you are considering the last visit).

Return the result table ordered by user_id.
```mysql
with cte as(
select *, 
ifnull(datediff(lead(visit_date,1) over(partition by user_id order by visit_date asc), visit_date),
       datediff('2021-1-1',visit_date)) as wd
from UserVisits
)

select user_id, max(wd) as biggest_window
from cte 
group by user_id 
order by user_id asc
```
### Leetcode 1699. Number of Calls Between Two Persons
Write an SQL query to report the number of calls and the total call duration between each pair of distinct persons (person1, person2) where person1 < person2.

Return the result table in any order.
```mysql 
with cte as (
select * from Calls 
where from_id<to_id
    union all
select to_id as from_id, from_id as to_id, duration
from Calls
where from_id>to_id
)

select from_id as person1, to_id as person2,
count(*) as call_count, sum(duration) as total_duration
from cte 
group by person1, person2
```
### Leetcode 1693. Daily Leads and Partners
Write an SQL query that will, for each date_id and make_name, return the number of distinct lead_id's and distinct partner_id's.

Return the result table in any order.

```mysql
select date_id, make_name, count(distinct lead_id) as unique_leads,
count(distinct partner_id) as unique_partners
from DailySales
group by date_id, make_name
```
### Leetcode 1683. Invalid Tweets
Write an SQL query to find the IDs of the invalid tweets. The tweet is invalid if the number of characters used in the content of the tweet is strictly greater than 15.

Return the result table in any order.
```mysql
select tweet_id from Tweets
where length(content)>15
```
### Leetcode 1677. Product's Worth Over Invoices
Write an SQL query that will, for all products, return each product name with total amount due, paid, canceled, and refunded across all invoices.

Return the result table ordered by product_name.
```mysql
select b.name, ifnull(sum(a.rest),0) as rest, ifnull(sum(a.paid),0) as paid, 
ifnull(sum(a.canceled),0) as canceled, ifnull(sum(a.refunded),0) as refunded
from Invoice as a
right join Product as b
on a.product_id=b.product_id
group by name
order by name asc
```
### Leetcode 1667. Fix Names in a Table
Write an SQL query to fix the names so that only the first character is uppercase and the rest are lowercase.

Return the result table ordered by user_id.
```mysql
select user_id, concat(upper(left(name,1)),lower(substr(name,2))) as name
from Users
order by user_id
```
### Leetcode 1661. Average Time of Process per Machine
There is a factory website that has several machines each running the same number of processes. Write an SQL query to find the average time each machine takes to complete a process.

The time to complete a process is the 'end' timestamp minus the 'start' timestamp. The average time is calculated by the total time to complete every process on the machine divided by the number of processes that were run.

The resulting table should have the machine_id along with the average time as processing_time, which should be rounded to 3 decimal places.
```mysql
select a.machine_id, round(avg(b.timestamp-a.timestamp),3) as processing_time 
from Activity as a
join Activity as b
on a.machine_id=b.machine_id and a.process_id=b.process_id
where a.activity_type='start' and b.activity_type='end'
group by machine_id
```
### Leetcode 1651. Hopper Company Queries III
Write an SQL query to compute the average_ride_distance and average_ride_duration of every 3-month window starting from January - March 2020 to October - December 2020. Round average_ride_distance and average_ride_duration to the nearest two decimal places.

The average_ride_distance is calculated by summing up the total ride_distance values from the three months and dividing it by 3. The average_ride_duration is calculated in a similar way.

Return the result table ordered by month in ascending order, where month is the starting month's number (January is 1, February is 2, etc.).
```mysql
with recursive cte as (
select 1 as month
union all
select month+1 from cte where month<12
),
cte1 as(
select month(b.requested_at) as month, 
    sum(a.ride_distance) as distance, sum(a.ride_duration) as duration
from AcceptedRides as a 
left join Rides as b
on a.ride_id=b.ride_id
where year(b.requested_at)=2020
group by month
)

select a.month, 
round(avg(ifnull(b.distance,0)) over(order by month rows between current row and 2 following),2) as average_ride_distance,
round(avg(ifnull(b.duration,0)) over(order by month rows between current row and 2 following),2) as 
average_ride_duration 
from cte as a
left join cte1 as b
on a.month=b.month
order by month
limit 10
```
### Leetcode 1645. Hopper Company Queries II
Write an SQL query to report the percentage of working drivers (working_percentage) for each month of 2020 

Note that if the number of available drivers during a month is zero, we consider the working_percentage to be 0.

Return the result table ordered by month in ascending order, where month is the month's number (January is 1, February is 2, etc.). Round working_percentage to the nearest 2 decimal places.
```mysql
with recursive cte as (
select 1 as month
union all
select month+1 from cte where month<12
),
cte1 as (
select a.month, ifnull(count(b.driver_id),0) as ava_drivers
from cte as a
left join Drivers as b
on (a.month>=month(b.join_date) or year(b.join_date)<2020) and year(b.join_date)<2021
group by month
),
cte2 as(
select month(b.requested_at) as month, count(distinct a.driver_id) as active_drivers
from AcceptedRides as a
left join Rides as b
on a.ride_id=b.ride_id 
where year(b.requested_at)=2020
group by month
)

select a.month, 
(case when a.ava_drivers>0 then round(ifnull(b.active_drivers*100/a.ava_drivers,0),2) 
 else 0 end) as working_percentage 
from cte1 as a
left join cte2 as b
on a.month=b.month
order by month asc
```
### Leetcode 1635. Hopper Company Queries I
Write an SQL query to report the following statistics for each month of 2020:

The number of drivers currently with the Hopper company by the end of the month (active_drivers).
The number of accepted rides in that month (accepted_rides).
Return the result table ordered by month in ascending order, where month is the month's number (January is 1, February is 2, etc.).
```mysql
with recursive cte as (
select 1 as month
union all
select month+1 from cte
where month<12
),
cte1 as (
select a.month, ifnull(count(b.driver_id),0) as active_drivers
from cte as a
left join Drivers as b
on (a.month>=month(b.join_date) or year(b.join_date)<2020) and year(b.join_date)<=2020
group by month
),
cte2 as(
select month(b.requested_at) as month, count(a.ride_id) as accepted_rides
from AcceptedRides as a
left join Rides as b
on a.ride_id=b.ride_id and year(b.requested_at)=2020
group by month
)

select a.month, a.active_drivers, ifnull(b.accepted_rides,0) as accepted_rides
from cte1 as a
left join cte2 as b
on a.month=b.month
order by month
```
### Leetcode 1633. Percentage of Users Attended a Contest
Write an SQL query to find the percentage of the users registered in each contest rounded to two decimals.

Return the result table ordered by percentage in descending order. In case of a tie, order it by contest_id in ascending order.

```mysql
select contest_id, 
round(100*count(user_id)/(select count(user_id) from Users),2) as percentage
from Register
group by contest_id
order by percentage desc, contest_id asc
```

### Leetcode 1623. All Valid Triplets That Can Represent a Country
There is a country with three schools, where each student is enrolled in exactly one school. The country is joining a competition and wants to select one student from each school to represent the country such that:

member_A is selected from SchoolA,
member_B is selected from SchoolB,
member_C is selected from SchoolC, and
The selected students' names and IDs are pairwise distinct (i.e. no two students share the same name, and no two students share the same ID).
Write an SQL query to find all the possible triplets representing the country under the given constraints.

Return the result table in any order.
```mysql
select a.student_name as member_A,
b.student_name as member_B, c.student_name as member_C
from SchoolA as a
join SchoolB as b
on a.student_id!=b.student_id and a.student_name!=b.student_name
join SchoolC as c
on a.student_id!=c.student_id and a.student_name!=c.student_name
and b.student_id!=c.student_id and b.student_name!=c.student_name
```





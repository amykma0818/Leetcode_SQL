### Leetcode 1990. Count the Number of Experiments

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


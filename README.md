# MutualFunds
SQL Project using Mutual Funds Data
This SQL Project using Mutual Funds in India provides insights into the vast variety and depth of the various schemes offered by the many fundhouses

Dataset Source:https://www.kaggle.com/datasets/ravibarnawal/mutual-funds-india-detailed
Database used:Postgre SQL

------------------------------------------------------------------------------------------------

## Task 1:
### Add a column and populate it with successive values and make them primary key

alter table if exists mutualfunds.mfdata
    add column mfid real;
	
alter table mfdata
	add primary key (mfid);

create temp table tempids as
select scheme_name,row_number() over (order by scheme_name) +1000 as seq
from mfdata;
update mfdata mf
set mfid=t.seq
from tempids t
where mf.scheme_name=t.scheme_name;

------------------------------------------------------------------------------------------------

Task 2:
Retrieve all funds where the minimum SIP amount is less than 500 and the rating is 4 or higher.

select scheme_name,min_sip,rating from mfdata where min_sip<500 and rating >=4

------------------------------------------------------------------------------------------------

Task 3:
Find the average expense_ratio and sharpe for each category.

select amc_name, avg(exp_ratio) as AVGEXPR, avg(sharpe) as AVGSHARPE from mfdata
group by amc_name

------------------------------------------------------------------------------------------------

Task 4:
Top 3 Categories of MF

select amc_name, count(scheme_name) from mfdata  group by amc_name order by count(scheme_name) desc limit 3

select category, count(scheme_name) from mfdata group by category order by count(scheme_name) desc limit 3

select sub_category, count(scheme_name) from mfdata group by sub_category order by count(scheme_name) desc limit 3

------------------------------------------------------------------------------------------------

Task 5:
Fund House Fund Name ordered descending by last 5 year return

select amc_name,scheme_name,max(returns_one),max(returns_three),max(returns_five)
from mfdata group by amc_name,scheme_name 
Having max(returns_five) is not null
order by max(returns_five) desc;

------------------------------------------------------------------------------------------------

Task 6:
Classification using Case

select scheme_name,rating,
case
when rating <=2 then 'Low'
when rating =3 then 'Medium'
else 'High' END
as CategoryRating
from mfdata

------------------------------------------------------------------------------------------------

Task 7:
Funds where Fund Size is more than the average fund size

select amc_name,scheme_name,avg(fund_size) as AVGFundSize from mfdata as m1
group by amc_name, scheme_name
having avg(fund_size)>
(select avg(fund_size) from mfdata as m2
where m2.amc_name=m1.amc_name)

------------------------------------------------------------------------------------------------

Task 8:
Fourth Largest Fund

with cte as
(select scheme_name,fund_size,
ROW_NUMBER() over (order by fund_size desc) as AUMRNK
from mfdata)
select scheme_name,fund_size,AUMRNK from cte where AUMRNK=4

------------------------------------------------------------------------------------------------

Task 9:
Second Largest Fund within each AMC

with cte as
(select amc_name, scheme_name,fund_size,
RANK() over (partition by amc_name order by fund_size desc) as AUMRNK
from mfdata)
select amc_name,scheme_name,fund_size,AUMRNK from cte where AUMRNK=2

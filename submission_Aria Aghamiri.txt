/*

-----------------------------------------------------------------------------------------------------------------------------------
													    Guidelines
-----------------------------------------------------------------------------------------------------------------------------------

The provided document is a guide for the project. Follow the instructions and take the necessary steps to finish
the project in the SQL file			

-----------------------------------------------------------------------------------------------------------------------------------
                                                         Queries
                                
-----------------------------------------------------------------------------------------------------------------------------------*/
  use new_wheels;  
  
/*-- QUESTIONS RELATED TO CUSTOMERS
     [Q1] What is the distribution of customers across states?
     Hint: For each state, count the number of customers.*/
     
select 
  count(customer_id) as number_of_customer, 
  state 
from new_wheels.customer_t
Group by 2
order by 1 DESC;

-- ---------------------------------------------------------------------------------------------------------------------------------

/* [Q2] What is the average rating in each quarter?
-- Very Bad is 1, Bad is 2, Okay is 3, Good is 4, Very Good is 5.

Hint: Use a common table expression and in that CTE, assign numbers to the different customer ratings. 
      Now average the feedback for each quarter. 

Note: For reference, refer to question number 4. Week-2: mls_week-2_gl-beats_solution-1.sql. 
      You'll get an overview of how to use common table expressions from this question.*/

(SELECT 
	quarter_number,
	(SUM(CASE WHEN customer_feedback ='Very Good' THEN 5
        WHEN customer_feedback ='Good' THEN 4
        WHEN customer_feedback ='Okay' THEN 3
        WHEN customer_feedback ='Bad' THEN 2
        else 1
	end)) as "total_feedback",
    (AVG(CASE WHEN customer_feedback ='Very Good' THEN 5
        WHEN customer_feedback ='Good' THEN 4
        WHEN customer_feedback ='Okay' THEN 3
        WHEN customer_feedback ='Bad' THEN 2
        else 1
	end)) as "avg_feedback",
    (count(customer_feedback)) as "feedback_count"
 from order_t
 GROUP by quarter_number
 ORDER by quarter_number);
 
-- ---------------------------------------------------------------------------------------------------------------------------------

/* [Q3] Are customers getting more dissatisfied over time?

Hint: Need the percentage of different types of customer feedback in each quarter. Use a common table expression and
	  determine the number of customer feedback in each category as well as the total number of customer feedback in each quarter.
	  Now use that common table expression to find out the percentage of different types of customer feedback in each quarter.
      Eg: (total number of very good feedback/total customer feedback)* 100 gives you the percentage of very good feedback.
      
Note: For reference, refer to question number 4. Week-2: mls_week-2_gl-beats_solution-1.sql. 
      You'll get an overview of how to use common table expressions from this question.*/ 
     
SELECT 
	quarter_number,
	(SUM(customer_feedback='Very Good')/count(*)) as "very_good_perc", 
	(SUM(customer_feedback='Good')/count(*)) as "good_perc", 
	(SUM(customer_feedback='Okay')/count(*)) as "okay_perc", 
	(SUM(customer_feedback='Bad')/count(*)) as "bad_perc", 
	(SUM(customer_feedback='Very Bad')/count(*)) as "very_bad_perc"
from order_t 
GROUP by quarter_number
ORDER by quarter_number;

-- ---------------------------------------------------------------------------------------------------------------------------------

/*[Q4] Which are the top 5 vehicle makers preferred by the customer.

Hint: For each vehicle make what is the count of the customers.*/

select 
vehicle_maker,
(count(*)) as "total_orders"
from order_t 
inner join product_t on order_t.product_id = product_t.product_id
group by vehicle_maker
order by total_orders DESC
LIMIT 5;

-- ---------------------------------------------------------------------------------------------------------------------------------

/*[Q5] What is the most preferred vehicle make in each state?

Hint: Use the window function RANK() to rank based on the count of customers for each state and vehicle maker. 
After ranking, take the vehicle maker whose rank is 1.*/
-- product_t -- vehicle maker, product_id
-- order_t -- product_id, customer_id
-- customer_t -- customer_id, state

SELECT state, vehicle_maker

FROM (

    SELECT state, vehicle_maker,

           ROW_NUMBER() OVER (PARTITION BY state ORDER BY COUNT(*) DESC) AS ranking

    FROM customer_t cust

    JOIN order_t orde ON cust.customer_id = orde.customer_id

    JOIN product_t prod ON orde.product_id = prod.product_id

    GROUP BY state, vehicle_maker

) AS subquery

WHERE ranking = 1;

-- ---------------------------------------------------------------------------------------------------------------------------------

/*QUESTIONS RELATED TO REVENUE and ORDERS 

-- [Q6] What is the trend of number of orders by quarters?

Hint: Count the number of orders for each quarter.*/

select 
  count(order_id) as number_of_order, 
  quarter_number
from new_wheels.order_t
Group by 2
order by 1 DESC;

-- ---------------------------------------------------------------------------------------------------------------------------------

/* [Q7] What is the quarter over quarter % change in revenue? 

Hint: Quarter over Quarter percentage change in revenue means what is the change in revenue from the subsequent quarter to the previous quarter in percentage.
      To calculate you need to use the common table expression to find out the sum of revenue for each quarter.
      Then use that CTE along with the LAG function to calculate the QoQ percentage change in revenue.
*/

 with q_change as
 ( 
    select
	quarter_number,
    sum(vehicle_price * quantity - vehicle_price * discount) as total_revenue
from new_wheels.order_t
group by 1
order by 1
)
select
	quarter_number,
    total_revenue,
    lag(total_revenue) over(order by quarter_number) as previous_revenue,
    (total_revenue - lag(total_revenue) over(order by quarter_number))/lag(total_revenue) over(order by quarter_number) *100 as q_perc_change
from q_change;

-- ---------------------------------------------------------------------------------------------------------------------------------

/* [Q8] What is the trend of revenue and orders by quarters?

Hint: Find out the sum of revenue and count the number of orders for each quarter.*/

-- quantity, vehicle_price, discount, quarter 
-- order_t

select
	quarter_number,
    sum(vehicle_price * quantity - vehicle_price * discount) as total_revenue,
    count(order_id)
from new_wheels.order_t
group by 1
order by 1;

-- ---------------------------------------------------------------------------------------------------------------------------------

/* QUESTIONS RELATED TO SHIPPING 
    [Q9] What is the average discount offered for different types of credit cards?

Hint: Find out the average of discount for each credit card type.*/

select
credit_card_type,
AVG(discount) *100 as discount_avg
from customer_t
join order_t using(customer_id)
group by credit_card_type
order by discount_avg;

-- ---------------------------------------------------------------------------------------------------------------------------------

/* [Q10] What is the average time taken to ship the placed orders for each quarters?
	Hint: Use the dateiff function to find the difference between the ship date and the order date.
*/

select
quarter_number,
AVG(datediff(ship_date, order_date)) as avg_time_to_ship
from new_wheels.order_t
group by quarter_number
order by avg_time_to_ship;



-- queries for business overview     
          
     create table ratings as (
 select *,
        CASE WHEN customer_feedback ='Very Good' THEN 5
        WHEN customer_feedback ='Good' THEN 4
        WHEN customer_feedback ='Okay' THEN 3
        WHEN customer_feedback ='Bad' THEN 2
        else 1
        end feedback
    FROM new_wheels.order_t);	

     
select
    sum(vehicle_price * quantity - vehicle_price * discount) as total_revenue
from new_wheels.order_t;


select
	count(distinct order_id) as total_orders
from new_wheels.order_t;

select
	count(distinct customer_id) as total_customers
from new_wheels.order_t;

select
	avg(feedback) as avg_rating
from new_wheels.ratings;

select
    quarter_number,
    sum(vehicle_price * quantity - vehicle_price * discount) as total_revenue
from new_wheels.order_t
group by quarter_number;

select
	quarter_number,
    count(distinct order_id) as order_count
from new_wheels.order_t
group by quarter_number;


select
AVG(datediff(ship_date, order_date)) as time_to_ship
from new_wheels.order_t;

use new_wheels;

select
count(customer_feedback) as total_cust_feedback
from order_t;

WITH customer_feedback AS (
SELECT 
        quarter_number,
        CASE WHEN customer_feedback ='Very Good' THEN 5
        WHEN customer_feedback ='Good' THEN 4
        WHEN customer_feedback ='Okay' THEN 3
        WHEN customer_feedback ='Bad' THEN 2
        else 1
        end feedback
    FROM new_wheels.order_t)	
Select quarter_number, 
		feedback,
        count(feedback) as feedback_count,
	    sum(feedback) as total_feedback
From customer_feedback
Group by 1,2
order by 1;

-- --------------------------------------------------------Done----------------------------------------------------------------------
-- ----------------------------------------------------------------------------------------------------------------------------------




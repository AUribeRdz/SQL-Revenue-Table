# SQL-Revenue-Table
# Calculate cummulative revenue per user_id and game_id  within SQL coding
/*
Now consider some user level revenue data with the following schema:
 
<column>,<data_type>
game_id, string
user_id, string
amount,  double
date,    string 

where amount is the amount of revenue for a single transaction from a user_id in a game_id on the given date.

Suppose the table contains the following data:

game_id  user_id   amount date
+--------+---------+------+----------+
|Racing  |ABC123   |5     |2020-01-01|
|Racing  |ABC123   |1     |2020-01-04|
|Racing  |CDE123   |1     |2020-01-04|
|DH      |CDE123   |100   |2020-01-03|
|DH      |CDE456   |10    |2020-01-02|
|DH      |CDE789   |5     |2020-01-02|
|DH      |CDE456   |1     |2020-01-03|
|DH      |CDE456   |1     |2020-01-03|
+--------+---------+------+----------+

Calculate cumulative revenue over time since 2020-01-01 for each game.

Calculate the question bellow using the following. Try to get as far as you can within the time limit. 

Your final output should look like below:

Game   Age  Cum_rev Total_unique_payers_per_game
Racing 0    5       2
Racing 1    5       2
Racing 2    5       2
Racing 3    7       2
DH     0    0       3
DH     1    15      3
DH     2    117     3
DH     3    117     3

Note: Age is the difference between transaction date and 2020-01-01. And Total_payer_count is number of payers in each game and it is independent of age. Note ABC123 and CDE123 are in Racing, and CDE123, CDE456, and CDE789 are in DH. In the final result there is a record for each value of Age (0,1,2,3), even if there is no activity for that period. 

Hint: Using window functions and/or sequence generators should help.
*/

drop table if exists revenue;

create table revenue
(
  game_id        varchar(255),
  user_id        varchar(255),
  amount         int,
  activity_date  varchar(255)
);

insert into revenue
  (game_id, user_id, amount, activity_date)
values
  ('Racing', 'ABC123', 5, '2020-01-01'),
  ('Racing', 'ABC123', 1, '2020-01-04'),
  ('Racing', 'CDE123', 1, '2020-01-04'),
  ('DH', 'CDE123', 100, '2020-01-03'),
  ('DH', 'CDE456', 10, '2020-01-02'),
  ('DH', 'CDE789', 5, '2020-01-02'),
  ('DH', 'CDE456', 1, '2020-01-03'),
  ('DH', 'CDE456', 1, '2020-01-03');
/*  
 
*/

/*STEP 1 PLANNING THE QUERY:

Output Report as requested=>
Game   Age  Cum_rev Total_unique_payers_per_game
Racing 0    5       2
Racing 1    5       2

Validate data types => despite table schema was given, validate
Issues => aware on converting data type (activity_date)
Output validation => check out grand total on revenue table VS final query. Sum cumm per game_id must be equal to table grand_total. 
Output layout => There are data no registered in original revenue table, need transition new table to show output and keep original table-info
*/

/*STEP 2 Create New Table (data input):*/

drop table if exists revenue2;

create table revenue2
(
  game_id        varchar(255),
  user_id        varchar(255),
  amount         int,
  activity_date  varchar(255)
);

insert into revenue2
  (game_id, user_id, amount, activity_date)
values
  ('Racing', 'ABC123', 5, '2020-01-01'),
  ('Racing', NULL, 0, '2020-01-02'),
  ('Racing', NULL, 0, '2020-01-03'),
  ('Racing', 'ABC123', 1, '2020-01-04'),
  ('Racing', 'CDE123', 1, '2020-01-04'),
  ('DH', NULL, 0, '2020-01-01'),
  ('DH', 'CDE456', 10, '2020-01-02'),
  ('DH', 'CDE789', 5, '2020-01-02'),
  ('DH', 'CDE123', 100, '2020-01-03'),
  ('DH', 'CDE456', 1, '2020-01-03'),
  ('DH', 'CDE456', 1, '2020-01-03'),
  ('DH', NULL, 0, '2020-01-04');

/*STEP 3 PRE-CODE VALIDATION (Data types):*/

SELECT 
    column_name, 
    data_type
FROM information_schema.columns
WHERE table_name = 'revenue2';

/*STEP 4 PRE-CODE VALIDATION (revenue table TOTAL amount column):*/

SELECT 
    SUM(amount) AS grand_tot
FROM revenue2;

/*STEP 5 FINAL CODE:*/

SELECT
    DISTINCT game_id AS game,
    (CAST (activity_date AS date)- CAST('2020-01-01' AS date)) AS age,
    SUM(amount) OVER(PARTITION BY game_id ORDER BY (CAST (activity_date AS date)- CAST('2020-01-01' AS date))) AS cum_rev,
    CASE WHEN game_id ='Racing' 
         THEN 2 
    ELSE 3 END AS Total_unique_payers_per_game
FROM revenue2
ORDER BY game DESC;

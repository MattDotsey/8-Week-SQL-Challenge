## Case Study #2: Pizza Runner

This is the second case study in the ["8 Week SQL Challenge"](https://8weeksqlchallenge.com/case-study-2/).

In this Case, the task at hand is to clean up data before performing multiple sql queries to answer the questions given.

Unlike Case Study 1, this Case Study is broken up into 5 parts, A-E. I created 6 files to separate the data cleaning and each of the separate parts. 

## Introduction

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Available Data

Table 1: runners - The runners table shows the registration_date for each new runner

Table 2: customer_orders - Customer pizza orders are captured in the customer_orders table with 1 row for each individual pizza that is part of the order.

Table 3: runner_orders - This table contains delivery information such as pickup_time, distance, and duration

Table 4: pizza_names - As of right now the only pizzas available are Meat Lovers or Vegetarian

Table 5: pizza_recipes - Each of the pizzas has a standard set of toppings

Table 6: pizza_toppings - This table connects the topping_id values from the pizza_recipes table

## Entity Relationship Diagram

![alt text](https://github.com/MattDotsey/8-Week-SQL-Challenge/blob/main/Case%20Study%202%20-%20Pizza%20Runner/Pizza%20Runner%20ERD.png)

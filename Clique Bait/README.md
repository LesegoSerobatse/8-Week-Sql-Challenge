![Alt text](<clique bait pics/cb.png>)

# Introduction

Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry!

In this case study - you are required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.


# Available Data

For this case study there is a total of 5 datasets which you will need to combine to solve all of the questions.


#### Users

Customers who visit the Clique Bait website are tagged via their cookie_id.

![Alt text](<clique bait pics/cb1.png>)


#### Events

Customer visits are logged in this events table at a cookie_id level and the event_type and page_id values can be used to join onto relevant satellite tables to obtain further information about each event.

The sequence_number is used to order the events within each visit.

![Alt text](<clique bait pics/cb2.png>)



#### Event Identifier

The event_identifier table shows the types of events which are captured by Clique Bait’s digital data systems.

![Alt text](<clique bait pics/cb3.png>)


#### Campaign Identifier

This table shows information for the 3 campaigns that Clique Bait has ran on their website so far in 2020.

![Alt text](<clique bait pics/cb4.png>)


#### Page Hierarchy

This table lists all of the pages on the Clique Bait website which are tagged and have data passing through from user interaction events.

![Alt text](<clique bait pics/cb5.png>)



# Case Study Questions

Using the following DDL schema details to create an ERD for all the Clique Bait datasets.

Click here to access the DB Diagram tool to create the ERD.

![Alt text](<clique bait pics/cb6.png>)


# 2. Digital Analysis

Using the available datasets - answer the following questions using a single query for each one:

1. How many users are there?
2. How many cookies does each user have on average?
3. What is the unique number of visits by all users per month?
4. What is the number of events for each event type?
5. What is the percentage of visits which have a purchase event?
6. What is the percentage of visits which view the checkout page but do not have a purchase event?
7. What are the top 3 pages by number of views?
8. What is the number of views and cart adds for each product category?
9. What are the top 3 products by purchases?


# 3. Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

   - How many times was each product viewed?
   - How many times was each product added to cart?
   - How many times was each product added to a cart but not purchased (abandoned)?
   - How many times was each product purchased?
  
Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions:

1. Which product had the most views, cart adds and purchases?
2. Which product was most likely to be abandoned?
3. Which product had the highest view to purchase percentage?
4. What is the average conversion rate from view to cart add?
5. What is the average conversion rate from cart add to purchase?


# 3. Campaigns Analysis

Generate a table that has 1 single row for every unique visit_id record and has the following columns:

   - user_id
   - visit_id
   - visit_start_time: the earliest event_time for each visit
   - page_views: count of page views for each visit
   - cart_adds: count of product cart add events for each visit
   - purchase: 1/0 flag if a purchase event exists for each visit
   - campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
    impression: count of ad impressions for each visit
   - click: count of ad clicks for each visit
   - (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

Some ideas you might want to investigate further include:

   - Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
   - Does clicking on an impression lead to higher purchase rates?
   - What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
   - What metrics can you use to quantify the success or failure of each campaign compared to eachother?























# Amazon delivery
In this project, we will examine data related to Amazon deliveries. The dataset includes information about agent ratings, geographical locations, agent ages, types of vehicles used, current weather conditions, and road conditions. Based on this data, we will analyze various trends that impact delivery performance.

**Columns:**
- **Order_ID**: Unique identifier for the order.
- **Agent_Age**: Age of the delivery agent.
- **Agent_Rating**: Rating of the delivery agent.
- **Store_Latitude**: Latitude of the store.
- **Store_Longitude**: Longitude of the store.
- **Drop_Latitude**: Latitude of the delivery location.
- **Drop_Longitude**: Longitude of the delivery location.
- **Order_Date**: Date when the order was placed.
- **Order_Time**: Time when the order was placed.
- **Pickup_Time**: Time when the order was picked up for delivery.
- **Weather**: Weather conditions during the delivery.
- **Trafic**: Traffic conditions during the delivery.
- **Vehicle**: Type of vehicle used for the delivery.
- **Area**: Area where the delivery took place.
- **Delivery_Time**: Time taken to deliver the order.
- **Category**: Category of the order.

## Initial Queries

After uploading the dataset, column names were loaded in the first row, so we will change the first row into column names.

```sql
SELECT * FROM amazon_delivery LIMIT 1;

ALTER TABLE amazon_delivery 
    CHANGE `COL 1` `Order_ID` VARCHAR(40),
    CHANGE `COL 2` `Agent_Age` INT,
    CHANGE `COL 3` `Agent_Rating` FLOAT,
    CHANGE `COL 4` `Store_Latitude` FLOAT,
    CHANGE `COL 5` `Store_Longitude` FLOAT,
    CHANGE `COL 6` `Drop_Latitude` FLOAT,
    CHANGE `COL 7` `Drop_Longitude` FLOAT,
    CHANGE `COL 8` `Order_Date` DATE,
    CHANGE `COL 9` `Order_Time` TIME,
    CHANGE `COL 10` `Pickup_Time` TIME,
    CHANGE `COL 11` `Weather` TEXT(20),
    CHANGE `COL 12` `Trafic` TEXT(20),
    CHANGE `COL 13` `Vehicle` VARCHAR(30),
    CHANGE `COL 14` `Area` TEXT(30),
    CHANGE `COL 15` `Delivery_Time` INT,
    CHANGE `COL 16` `Category` TEXT;

DELETE FROM amazon_delivery LIMIT 1;
```


* Performed a few initial queries to check the data and its accuracy; 

```sql
SELECT COUNT(*) FROM amazon_delivery;
```

To check if `Order_ID` has unique values:

```sql
SELECT Order_ID, COUNT(*)
FROM amazon_delivery
GROUP BY Order_ID
HAVING COUNT(*) > 1;
```
* The query results indicated that each record is unique. *

*Data validation check:*
```sql
SELECT
    `Agent_Age`,
    `Agent_Rating`,
    `Weather`,
    `Trafic`,
    `Vehicle`,
    `Area`,
    `Delivery_Time`,
    `Category`,
    COUNT(*) AS Unique_value
FROM
    amazon_delivery
GROUP BY
    `Agent_Age`,
    `Agent_Rating`,
    `Weather`,
    `Trafic`,
    `Vehicle`,
    `Area`,
    `Delivery_Time`,
    `Category`;
```

Agent_Rating has values of 0 and Weather has values of NaN.

```sql
-- Checking which values are 0 in Agent_Rating
SELECT * FROM amazon_delivery WHERE `Agent_Rating` = 0;

-- Checking which values are 'NaN' in Weather
SELECT * FROM amazon_delivery WHERE Weather = 'NaN';

-- Updating Agent_Rating to the average value of the Agent_Rating column
-- First, calculating the average Agent_Rating excluding 0 values
SELECT ROUND(AVG(Agent_Rating), 1) AS avg_rating
FROM amazon_delivery
WHERE Agent_Rating != 0;

-- Updating the values
UPDATE amazon_delivery
SET Agent_Rating = (SELECT AVG(Agent_Rating) FROM amazon_delivery WHERE Agent_Rating != 0)
WHERE Agent_Rating = 0;
```

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/86144f93-6797-4d24-ad80-276675ddf87b)

*We replace the NaN value with the most frequently occurring value* 
```sql
SELECT `Weather`, COUNT(*) AS cnt
FROM amazon_delivery
WHERE `Weather` != 'NaN'
GROUP BY `Weather`
ORDER BY cnt DESC
LIMIT 1;
```

"FOG" is the most frequently chosen option.

```sql
UPDATE amazon_delivery
SET `Weather` = 'Fog'
WHERE `Weather` = 'NaN';
```

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/fd72b58b-8406-4e71-8533-a91323c3a6be)

# Data Analysis
**Once we have completed the data cleansing stage, we can proceed to the initial analysis.**

```sql
--Distribution by age;
SELECT `Agent_Age`, COUNT(*) AS Result FROM amazon_delivery GROUP BY `Agent_Age`;
```

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/19ad98f5-e942-4eae-9d5b-4c38dbf41858)

Most agents are in the age range of 20-39 years. However, the fewest are aged 15 and 50 years.

```sql
--Agent rating by age 
SELECT `Agent_Age`, `Agent_Rating`, COUNT(*) AS Result
FROM amazon_delivery
GROUP BY `Agent_Age`
HAVING `Agent_Rating` > 4.5
ORDER BY `Agent_Age`;
```

![obraz](https://github.com/user-attachments/assets/b3f64756-ceea-4ab0-88e4-bff90c79343c)

he majority of agents who have a rating higher than 4.5 are in the age range older than 24 years.

```sql
-- We will check which type of vehicle is chosen most frequently and what is the average delivery time by vehicle.
SELECT
    Vehicle,
    ROUND(AVG(Delivery_Time),2) AS Average_Delivery_Time,
    COUNT(*) AS Vehicle_Count
FROM
    amazon_delivery
GROUP BY
    Vehicle
ORDER BY
    Vehicle_Count DESC;
```

![obraz](https://github.com/user-attachments/assets/2f8aaf2a-7d4e-4198-b83e-af6520e0b6d4)

The most frequently chosen vehicle is the motorcycle. However, the best delivery time is achieved by the scooter and van.


```sql
-- The most frequent weather condition. 
SELECT `Weather`, COUNT(*) AS Weather_Count FROM amazon_delivery GROUP BY `Weather`;

-- The most frequent traffic condition. 
SELECT `Trafic`, COUNT(*) AS Trafic_Count FROM amazon_delivery GROUP BY `Trafic`;

-- The most frequently ordered categories.;
SELECT `Category`, COUNT(*) AS Category_Count FROM amazon_delivery GROUP BY `Category`;
```
![obraz](https://github.com/user-attachments/assets/4ffa1f59-cb49-4b7e-9ea5-91a6ff92a2a8) ![obraz](https://github.com/user-attachments/assets/b60f7069-7550-46f4-b80d-90698623c39d) ![obraz](https://github.com/user-attachments/assets/e6a9b6c9-539e-4101-b382-b1b8ec24ba60)

```sql
-- Average delivery time due to traffic and area.
SELECT `Trafic`, `Area`, ROUND(AVG(`Delivery_Time`), 2) as Avg_Delivery_Time 
FROM amazon_delivery 
GROUP BY `Trafic`, `Area` 
ORDER BY Avg_Delivery_Time;

-- The correlation between weather and traffic on the average delivery time.
SELECT `Trafic`, `Weather`, ROUND(AVG(`Delivery_Time`), 2) AS Avg_Delivery_Time
FROM `amazon_delivery`
GROUP BY `Trafic`, `Weather`
HAVING Avg_Delivery_Time > 120
ORDER BY Avg_Delivery_Time;


```
![obraz](https://github.com/user-attachments/assets/adc670ca-e3df-462e-a71a-131c7f17c3de)

The area with the longest average delivery time is the semi-urban area with high, jam, and medium traffic.

![obraz](https://github.com/user-attachments/assets/edc83350-cdcd-4f34-a220-3d80b2907e5c)

Orders that had an average delivery time of more than 120 correlated with adverse weather conditions.

```sql
-- The most common combinations of weather and traffic conditions for deliveries that exceeded the average delivery time.

WITH AverageDelivery AS (
    SELECT AVG(`Delivery_Time`) as Avg_Delivery_Time
    FROM amazon_delivery
)
SELECT `Weather`, `Trafic`, COUNT(*) as Occurrences
FROM amazon_delivery, AverageDelivery
WHERE `Delivery_Time` > `Avg_Delivery_Time`
GROUP BY `Weather`, `Trafic`
ORDER BY Occurrences DESC;
```
![obraz](https://github.com/user-attachments/assets/99cc95a5-5b79-452f-8d5d-2ea5e0e89844)

DO ANALIZY

Średnia ocena agenta dla różnych typów pojazdów:
SELECT `Vehicle`, ROUND(AVG(`Agent_Rating`),2) AS Avg_Agent_Rating FROM amazon_delivery GROUP BY `Vehicle`;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/f0560125-75be-416b-b0bd-0bb7f47b37be)

Analiza korelacji między czasem dostawy a oceną agenta:
SELECT `Agent_Rating`, ROUND(AVG(Delivery_Time),2) as Avg_Delivery_Time
FROM amazon_delivery
GROUP BY `Agent_Rating`
ORDER BY `Avg_Delivery_Time`;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/e98b04bc-d260-4c19-af21-950bcca54cbe)

Segmentacja klientów na podstawie średniego czasu dostawy i oceny agenta: ogarnij to ! 









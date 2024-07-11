# Amazon-delivery

Po wgraniu dataset nazwy kolumn wczytały się w pierwszym wierszu więc zamieniamy pierwszy wiersz na nazwy kolumn. 

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


Warto uzyskać pewne statystyki dotyczące danych, np. liczba wierszy:
SELECT COUNT(*) FROM nazwa_tabeli;

Sprawdzam czy Order_ID ma unikalne wartości 

SELECT Order_ID, COUNT(*)
FROM amazon_delivery
GROUP BY Order_ID
HAVING COUNT(*) > 1;

Sprawdzenia poprawności danych :
SELECT
    `Agent_Age`,
    `Agent_Rating`,
    `Weather`,
    `Trafic`,
    `Vehicle`,
    `Area`,
    `Delivery_Time`,
    `Category`,
    COUNT(*) as Unique_value
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

Agent_Raiting ma wartości 0 a weather wartości Nan 

Sprawdzam które wartości mają 0 oraz nan 
SELECT * FROM amazon_delivery WHERE `Agent_Rating` = 0;

SELECT *
FROM amazon_delivery
WHERE Weather = 'NaN';

Agent_Raiting zamieniamy na średnią wartość kolumny agent_rating
Najpierw obliczamy średnią wartość agent_rating ignorując wartość 0;

SELECT ROUND(AVG(Agent_Rating),1) AS avg_rating
FROM amazon_delivery
WHERE Agent_Rating != 0;

Dajemy updaty tej liczby : 
UPDATE amazon_delivery
SET Agent_Rating = (SELECT AVG(Agent_Rating) FROM amazon_delivery WHERE Agent_Rating != 0)
WHERE Agent_Rating = 0;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/86144f93-6797-4d24-ad80-276675ddf87b)

Zamieniamy wartość NaN na najczęściej powtarzająca się wartość 
SELECT `Weather`, COUNT(*) AS cnt
FROM amazon_delivery
WHERE `Weather` != 'NaN'
GROUP BY `Weather`
ORDER BY cnt DESC
LIMIT 1;

Najczęściej wybierany jest FOG

UPDATE amazon_delivery
SET `Weather` = 'Fog'
WHERE `Weather` = 'NaN';

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/fd72b58b-8406-4e71-8533-a91323c3a6be)

Dystrybucja ze względu na wiek;
SELECT `Agent_Age`, COUNT(*) AS Result FROM amazon_delivery GROUP BY `Agent_Age`;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/19ad98f5-e942-4eae-9d5b-4c38dbf41858)

Najmniej mamy agentów wieku 15 i 50 lat

Sprawdzamy jaka jest ocena agenta ze względu na wiek; 
SELECT `Agent_Age`,`Agent_Rating`, COUNT(*) AS Result FROM amazon_delivery GROUP BY `Agent_Age` ORDER BY `Agent_Rating`;
![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/cc0e8efd-11b2-4b89-85db-d5cd21deac7e)

Średni czas dostawy ze względu na pojazd 
SELECT `Vehicle`, ROUND(AVG(`Delivery_Time`),2) AS Avg_Delivery_Time FROM amazon_delivery GROUP BY `Vehicle`;
![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/91ab5008-5d73-4db4-b274-48fced37eca1)


Jaka najczęściej występuje pogoda :
SELECT `Weather`, COUNT(*) FROM amazon_delivery GROUP BY `Weather`;
![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/2cf9e821-ff9b-4fc5-9df2-f41e11643a65)

Najczęściej występujący pojazd 
SELECT `Vehicle`, COUNT(*) FROM amazon_delivery GROUP BY `Vehicle`;

Najczęściej występujący ruch drogowy; 
SELECT `Trafic`, COUNT(*) FROM amazon_delivery GROUP BY `Trafic`;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/8fc67336-feea-476b-9fef-5def83bff2e6)

Zamawiane kategorie; 
SELECT `Category`, COUNT(*) FROM amazon_delivery GROUP BY `Category`;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/3033b02a-a0b1-4e47-b9bc-c5fa0ac84a8d)

Średni czas dostawy ze względu na kategorię; 
SELECT `Category`, AVG(`Delivery_Time`) AS Avg_Delivery_Time FROM amazon_delivery GROUP BY `Category` ORDER BY Avg_Delivery_Time;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/2e2c680b-f7ae-4409-8d26-58de24e743c3)

Średnia ocena agenta dla różnych typów pojazdów:
SELECT `Vehicle`, ROUND(AVG(`Agent_Rating`),2) AS Avg_Agent_Rating FROM amazon_delivery GROUP BY `Vehicle`;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/f0560125-75be-416b-b0bd-0bb7f47b37be)

Czas dostawy w zależności od stanu ruchu drogowego:
SELECT `Trafic`, ROUND(AVG(`Delivery_Time`), 2) as Avg_Delivery_Time FROM amazon_delivery GROUP BY `Trafic`;
![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/6c30d436-c916-4a12-ad8a-5f010b6f7ed0)

Średni czas dostawy dla różnych typów obszarów:
SELECT `Area`, ROUND(AVG(`Delivery_Time`),2) AS Avg_Delivery_Time FROM amazon_delivery GROUP BY `Area`;
![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/97108c47-7bf4-4017-b0a6-71a3fd3e93d9)



Analiza wpływu pogody i ruchu drogowego na czas dostawy:

SELECT `Weather`, `Trafic`, AVG(`Delivery_Time`) as Avg_Delivery_Time
FROM amazon_delivery
GROUP BY `Weather`, `Trafic`
ORDER BY `Avg_Delivery_Time` DESC;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/d9db94d4-a6e7-40bf-8e41-fd3179ae10f3)

najczęściej występujące kombinacje pogody i stanu ruchu dla dostaw, które przekroczyły średni czas dostawy:

WITH AverageDelivery AS (
    SELECT AVG(`Delivery_Time`) as Avg_Delivery_Time
    FROM amazon_delivery
)
SELECT `Weather`, `Trafic`, COUNT(*) as Occurrences
FROM amazon_delivery, AverageDelivery
WHERE `Delivery_Time` > `Avg_Delivery_Time`
GROUP BY `Weather`, `Trafic`
ORDER BY Occurrences DESC;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/8b5954d2-6832-48cf-b72b-30a1a3d10e81)

Analiza korelacji między czasem dostawy a oceną agenta:
SELECT `Agent_Rating`, ROUND(AVG(Delivery_Time),2) as Avg_Delivery_Time
FROM amazon_delivery
GROUP BY `Agent_Rating`
ORDER BY `Avg_Delivery_Time`;

![obraz](https://github.com/biku89/Amazon-delivery/assets/169537978/e98b04bc-d260-4c19-af21-950bcca54cbe)

Segmentacja klientów na podstawie średniego czasu dostawy i oceny agenta: ogarnij to ! 









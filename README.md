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

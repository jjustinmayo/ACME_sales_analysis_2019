# ACME_Sales_Analysis_2019

## Overall sales performance for 2019 

```sql
-- Total Revenue
SELECT 
	SUM(`Price Each` * `Quantity Ordered`) as `Total Revenue`
FROM acme_sales.acme_combined_sales_2019

-- Total Quantity Orderedd
SELECT 
	SUM(`Quantity Ordered`) as `Total Quantity Ordered`
FROM acme_sales.acme_combined_sales_2019

-- Average Product Price
SELECT 
	AVG(`Price Each`) as `Average Price`
FROM acme_sales.acme_combined_sales_2019
```


| Total Revenue       |
| ------------------- |
|  $       34,465,538 |

| Total Quantity Ordered |
| ---------------------- |
|         208,812        |

| Average Price      |
| ------------------ |
|  $          184.52 |


## Product Analysis

#### Product Sales Breakdown

```sql
SELECT 
	Product, 
	SUM(`Price Each` * `Quantity Ordered`) AS 'Total Revenue',
    SUM(`Price Each` * `Quantity Ordered`) / SUM(`Quantity Ordered`) AS 'Price',
    SUM(`Quantity Ordered`) AS 'Total Quantity Ordered'
FROM acme_sales.acme_combined_sales_2019
GROUP BY Product 
ORDER BY CASE -- Since the output of 'Total Revenue' is a text, we use a CASE WHEN clause to convert each value to SIGNED (64-bit integer) and then order DESC.
	WHEN SUM(`Price Each` * `Quantity Ordered`) >= 0 THEN CAST(SUM(`Price Each` * `Quantity Ordered`) AS SIGNED) ELSE NULL 
    END DESC;
```

| Product                    | Total Revenue         | Price           | Total Quantity Ordered |
| -------------------------- | --------------------- | --------------- | ---------------------- |
| Macbook Pro Laptop         |  $       8,032,500    |  $ 1,700.00     |           4,725        |
| iPhone                     |  $       4,792,900    |  $    700.00    |           6,847        |
| ThinkPad Laptop            |  $       4,127,959    |  $    999.99    |           4,128        |
| Google Phone               |  $       3,317,400    |  $    600.00    |           5,529        |
| 27in 4K Gaming Monitor     |  $       2,433,148    |  $    389.99    |           6,239        |
| 34in Ultrawide Monitor     |  $       2,352,898    |  $    379.99    |           6,192        |
| Apple Airpods Headphones   |  $       2,345,550    |  $    150.00    |         15,637         |
| Flatscreen TV              |  $       1,443,900    |  $    300.00    |           4,813        |
| Bose SoundSport Headphones |  $       1,342,866    |  $       99.99  |         13,430         |
| 27in FHD Monitor           |  $       1,131,075    |  $    149.99    |           7,541        |
| Vareebadd Phone            |  $          827,200   |  $    400.00    |           2,068        |
| 20in Monitor               |  $          453,819   |  $    109.99    |           4,126        |
| LG Washing Machine         |  $          399,600   |  $    600.00    |              666       |
| LG Dryer                   |  $          387,600   |  $    600.00    |              646       |
| Lightning Charging Cable   |  $          346,377   |  $       14.95  |         23,169         |
| USB-C Charging Cable       |  $          285,975   |  $       11.95  |         23,931         |
| Wired Headphones           |  $          246,083   |  $       11.99  |         20,524         |
| AA Batteries (4-pack)      |  $          106,042   |  $         3.84 |         27,615         |
| AAA Batteries (4-pack)     |  $             92,648 |  $         2.99 |         30,986         |

The Company's top three products which generated the most revenue included the Macbook Pro Laptop, iPhone, and ThinkPad Laptop. Combined, these products represented approximately 50% of the Company's total sales.

#### Complementary Product Analysis

```sql
SELECT 
    Product,
    COUNT(*) as Frequency
FROM acme_sales.acme_combined_sales_2019
-- from the subquery, we pull all individual products related to each 'Order Id' and rank how often each item has been purchased with another item in a single order.
WHERE `Order Id` IN 
    -- The subquery below is a table of all distinct rows that contain a duplicate 'Order Id,' which represents all IDs with multiple products purchased in a single order. 
	(SELECT 
		`Order Id`
	FROM acme_sales.acme_combined_sales_2019
	GROUP BY `Order Id` 
	HAVING COUNT(*) > 1
	) 
GROUP BY
	Product
ORDER BY 
	Frequency DESC
```

| Product                    | Frequency       |
| -------------------------- | --------------- |
| USB-C Charging Cable       |         2,025   |
| iPhone                     |         1,864   |
| Lightning Charging Cable   |         1,734   |
| Google Phone               |         1,633   |
| Wired Headphones           |         1,609   |
| Apple Airpods Headphones   |            926  |
| Bose SoundSport Headphones |            766  |
| AAA Batteries (4-pack)     |            757  |
| AA Batteries (4-pack)      |            730  |
| Vareebadd Phone            |            601  |
| 27in FHD Monitor           |            276  |
| 27in 4K Gaming Monitor     |            241  |
| 34in Ultrawide Monitor     |            232  |
| Macbook Pro Laptop         |            190  |
| ThinkPad Laptop            |            172  |
| Flatscreen TV              |            166  |
| 20in Monitor               |            153  |
| LG Washing Machine         |              28 |
| LG Dryer                   |              25 |

The table above summarizes items how frequently a product was purchased under a duplicate order_id, identifying a cross-sell. Based on this data, the top 5 cross-selling products under the average product price of $185 were USB-C Charging Cables, Lightning Charging Cables, Wired Headphones, Apple Airpods Headphones, and Bose SoundSport Headphones.  

#### Recommendations: 
* The Company should consider brick & mortar strategies which focus on complementary sales of charging cables and headphones. 
* The Company can also consider introducing complementary products such as desktops and speakers which can potentially improve cross-sales for monitors and TVs, respectively.

## Price Analysis & Seasonality

#### Price Analysis: 

```sql
SELECT
    temp.`Price Range`, 
	temp.`# of Product Categories`,
    SUM(`Quantity Ordered` * `Price Each`) as 'Total Revenue', 
    SUM(`Quantity Ordered`) as 'Total Quantity Ordered' 
FROM (

    -- created a temp table that sorts the data into 5 groups of prices ranges. Count each distinct product within each price range. 
    SELECT
        CASE
            WHEN `Price Each` < 49.99 THEN '$0-$49.99'
            WHEN `Price Each` BETWEEN 50 AND 149.99 THEN '$50-$149.99'
            WHEN `Price Each` BETWEEN 150 AND 499.99 THEN '$150-$499.99'
            WHEN `Price Each` BETWEEN 500 AND 999.99 THEN '$500-$999.99'
            ELSE 'over $1,000'
        END AS `Price Range`,
        COUNT(DISTINCT Product) as `# of Product Categories`
    FROM acme_sales.acme_combined_sales_2019
    GROUP BY `Price Range`
) as temp 


-- use a JOIN clause to join the schema with the temporary table by price ranges defined in the temporary table subquery. 
JOIN acme_sales.acme_combined_sales_2019 AS main ON temp.`Price Range` = 

    (CASE
        WHEN main.`Price Each` < 49.99 THEN '$0-$49.99'
        WHEN main.`Price Each` BETWEEN 50 AND 149.99 THEN '$50-$149.99'
        WHEN main.`Price Each` BETWEEN 150 AND 499.99 THEN '$150-$499.99'
        WHEN main.`Price Each` BETWEEN 500 AND 999.99 THEN '$500-$999.99'
        ELSE 'over $1,000'
    END)
GROUP BY temp.`Price Range`, temp.`# of Product Categories`
ORDER BY `Total Revenue`;
```

| Price Range  | \# of Product Categories | Total Revenue       | Total Quantity Ordered |
| ------------ | ------------------------ | ------------------- | ---------------------- |
| $500-$999.99 | 5                        |         13,025,459  |           17,816       |
| $150-$499.99 | 5                        |           9,402,696 |           34,949       |
| over $1,000  | 1                        |           8,032,500 |             4,725      |
| $50-$149.99  | 3                        |           2,927,759 |           25,097       |
| $0-$49.99    | 5                        |           1,077,124 |         126,225        |

Products within the price range $500-$999.99 generated the most revenue at $13 million. 

#### Recommendations: 
* The Company should continue focusing on the price range of $500 - $999.99, and strategize around the top performing products within this segment (i.e. sales, cross-selling opportunities).
* Investigated the opportunities in higher-end products priced over $1,000. Although this segment did not generate the highest revenue, it does have potential since only one product (Macbook Pro) was able to generate the sales of this segment. An opportunity exists to sell similar high-end, reputable products to increase the revenue/profitability/volume in this segment.  

#### Seasonality Analysis:

```sql
SELECT
    -- Since 'Order Date' is in text, we use a CASE WHEN and assign each number to the corresponding month. 
	CASE 
		WHEN Left(`Order Date`, 2) = 1 THEN 'January'
        WHEN Left(`Order Date`, 2) = 2 THEN 'February'
        WHEN Left(`Order Date`, 2) = 3 THEN 'March'
        WHEN Left(`Order Date`, 2) = 4 THEN 'April'
        WHEN Left(`Order Date`, 2) = 5 THEN 'May'
        WHEN Left(`Order Date`, 2) = 6 THEN 'June'
        WHEN Left(`Order Date`, 2) = 7 THEN 'July'
        WHEN Left(`Order Date`, 2) = 8 THEN 'August'
        WHEN Left(`Order Date`, 2) = 9 THEN 'September'
        WHEN Left(`Order Date`, 2) = 10 THEN 'October'
        WHEN Left(`Order Date`, 2) = 11 THEN 'November'
        WHEN Left(`Order Date`, 2) = 12 THEN 'December'
	END as 'Month',
	SUM(`Quantity Ordered` * `Price Each`) as 'Total Revenue',
    SUM(`Quantity Ordered`) as 'Total Quantity Ordered'
FROM acme_sales.acme_combined_sales_2019
GROUP BY 
	Month
ORDER BY
	`Total Revenue` DESC 
```

|    | Month     | Total Revenue     | Total Quantity Ordered |
| -- | --------- | ----------------- | ---------------------- |
| 1  | January   |  $      1,821,413 |         10,893         |
| 2  | February  |  $      2,200,078 |         13,431         |
| 3  | March     |  $      2,804,973 |         16,979         |
| 4  | April     |  $      3,389,218 |         20,536         |
| 5  | May       |  $      3,150,616 |         18,653         |
| 6  | June      |  $      2,576,280 |         15,234         |
| 7  | July      |  $      2,646,461 |         16,054         |
| 8  | August    |  $      2,241,083 |         13,429         |
| 9  | September |  $      2,094,466 |         13,091         |
| 10 | October   |  $      3,734,778 |         22,669         |
| 11 | November  |  $      3,197,875 |         19,769         |
| 12 | December  |  $      4,608,296 |         28,074         |

* The Company saw a significant lift in sales in Winter Q4 (Oct-Dec) and slightly in Spring Q2 (April-June).

#### Recommendations: 
* Capitalize on high selling months including April, October, December. Allocate marketing and promotional resources to maximize sales during these months.
* Similarly, for periods with relatively lower revenue such as January, February, August and September, allocate marketing and promotional resources to mitigate the seasonal dip in these months.
* Although inventory data is not included, it is worth pointing out regardless that inventory management during peak seasonal periods is critical. 

## Geographic Sales

```sql
SELECT
	SUBSTRING(`Purchase Address`, -8, 2) AS State,
	SUM(`Quantity Ordered` * `Price Each`) as 'Total Revenue'
FROM acme_sales.acme_combined_sales_2019
GROUP BY STATE
ORDER BY STATE ASC;
```

| State | Total Revenue       |
| ----- | ------------------- |
| CA    |  $     13,703,048   |
| GA    |  $       2,794,199  |
| MA    |  $       3,658,628  |
| ME    |  $          449,321 |
| NY    |  $       4,661,867  |
| OR    |  $       1,870,011  |
| TX    |  $       4,583,418  |
| WA    |  $       2,745,046  |

#### Recommendations: 
* Capitalize on high-revenue states including California, New York, and Texas by exploring market expansion opportunities and strengthening presence in these states (i.e. logistics, customer service, etc.).
* Investigate lower-revenue states such as Maine and Oregon for customer preferences, competition, etc.




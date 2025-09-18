# MySQL-Finance-and-Supply-chain-Analytics

## 📌 Project Short Info
SQL-based project focusing on financial and supply chain analytics using MySQL. Includes sales reports, market trends, customer insights, and forecast accuracy analysis to support business decision-making.

## 📖 Project Description
This project is built to analyze **finance and supply chain data** for Atliq Hardware using **MySQL**. It addresses performance issues caused by large Excel datasets by migrating analysis to a relational database.  

The project leverages advanced SQL techniques—joins, user-defined functions, stored procedures, views, temporary tables, and window functions—to create insights on **sales, market performance, and customer behavior**.  

Key reports include:  
- Product-wise and customer-wise sales reports  
- Monthly and yearly gross/net sales reports  
- Top-performing markets and customers  
- Supply chain performance analytics  
- Forecast accuracy comparison across years  

By combining technical SQL capabilities with analytical problem-solving, the project improves decision-making, enhances reporting efficiency, and deepens understanding of business performance.

---

## Prerequisite
* Download this file.

  [atliq_dim_table_gdb0041](https://drive.google.com/file/d/1cGcWzz9f0MsYpqhtgkTGluvR3si5Dga1/view?usp=drive_link)

 ## Problem Statement

Atliq Hardware has launched a project to address performance issues arising from the expansion of Excel files. To combat this challenge, the company wanted a team of data analysts who would leverage MySQL as their database and help analyze data, extract valuable insights and gain an understanding of the company's financial position. 
The primary objective is to enhance decison making and elevate overall performance.

**My course of action:** &nbsp; Assuming the role of a data analyst, I utilized MySQL as my database platform to tackle targeted inquiries regarding sales market dynamics, market analysis, and customer behavior. My primary objective is to unwrap insights that offer strategic direction and deepens the understanding of market trends and customer preferences.

I formulated SQL queries for the following requests. 
  1) Product Wise Sales Report for Croma in 2021
  2) Monthly Gross Sales Report for Croma 
  3) Yearly Gross Sales Report for Croma
  4) Top n Customers by Net Sales 
  5) Top Markets by Net Sales 
  6) Supply Chain Analytics Report for the year 2021 
  7) Forecast Accuracy Report (2020 vs 2021)



-----------------------------------------------------------------------------------------------------------------

***- Skills used :***

💡 Loading the data into MySQL and using JOINS

💡 Creating user defined functions

💡 Creating Stored Procedures

💡 Creating Views 

💡 Using Temporary tables and Common Table Expression (CTE)

💡 Applying Window Functions (OVER, PARTITION_BY, ROW_NUMBER, RANK, DENSE_RANK)


-----------------------------------------------------------------------------------------------------------------

***- Take a look at my presentation of queries:***

Link 👉 [Finance & Supply chain Analytics](https://drive.google.com/file/d/1ypMhMlMkEAwO17DReliJW_0E530Ps9z1/view?usp=sharing)


## User defined functions :
### Get fiscal month 
       CREATE DEFINER=`root`@`localhost` FUNCTION `get_fiscal_month`(calendar_date DATE) RETURNS int
         DETERMINISTIC
       BEGIN
       declare fiscal_month int;
       set fiscal_month = month(date_add(calendar_date, interval 4 month));
       RETURN fiscal_month ;
       END
       
### Get fiscal year
       CREATE DEFINER=`root`@`localhost` FUNCTION `get_fiscal_year`(calendar_date date) RETURNS int
         DETERMINISTIC
       BEGIN
       declare fiscal_year int;
       set fiscal_year = year(date_add(calendar_date,interval 4 month));
       RETURN fiscal_year;
       END

## Views :
### Gross sales
    CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
    VIEW `gross_sales` AS
        SELECT 
            `s`.`date` AS `date`,
            `s`.`product_code` AS `product_code`,
            `s`.`customer_code` AS `customer_code`,
            `p`.`product` AS `product`,
            `p`.`variant` AS `variant`,
            `s`.`sold_quantity` AS `sold_quantity`,
            `g`.`gross_price` AS `gross_price_per_item`,
            ROUND((`g`.`gross_price` * `s`.`sold_quantity`),
                    2) AS `gross_price_total`
        FROM
            ((`fact_sales_monthly` `s`
            JOIN `dim_product` `p` ON ((`s`.`product_code` = `p`.`product_code`)))
            JOIN `fact_gross_price` `g` ON (((`g`.`product_code` = `s`.`product_code`)
                AND (`g`.`fiscal_year` = `s`.`fiscal_year`))))
### Net sales
      CREATE 
          ALGORITHM = UNDEFINED 
          DEFINER = `root`@`localhost` 
          SQL SECURITY DEFINER
      VIEW `net_sales` AS
          SELECT 
              `sales_postin_discount`.`date` AS `date`,
              `sales_postin_discount`.`fiscal_year` AS `fiscal_year`,
              `sales_postin_discount`.`product_code` AS `product_code`,
              `sales_postin_discount`.`customer_code` AS `customer_code`,
              `sales_postin_discount`.`market` AS `market`,
              `sales_postin_discount`.`product` AS `product`,
              `sales_postin_discount`.`variant` AS `variant`,
              `sales_postin_discount`.`sold_quantity` AS `sold_quantity`,
              `sales_postin_discount`.`gross_price_per_item` AS `gross_price_per_item`,
              `sales_postin_discount`.`gross_price_total` AS `gross_price_total`,
              `sales_postin_discount`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`,
              `sales_postin_discount`.`net_invoice_sales` AS `net_invoice_sales`,
              `sales_postin_discount`.`post_invoice_discount_pct` AS `post_invoice_discount_pct`,
              ((1 - `sales_postin_discount`.`post_invoice_discount_pct`) * `sales_postin_discount`.`net_invoice_sales`) AS `net_sales`
          FROM
              `sales_postin_discount`
### Pre invoice discount sales
      CREATE 
          ALGORITHM = UNDEFINED 
          DEFINER = `root`@`localhost` 
          SQL SECURITY DEFINER
      VIEW `sales_prein_discount` AS
          SELECT 
              `s`.`date` AS `date`,
              `s`.`fiscal_year` AS `fiscal_year`,
              `s`.`product_code` AS `product_code`,
              `s`.`customer_code` AS `customer_code`,
              `c`.`market` AS `market`,
              `p`.`product` AS `product`,
              `p`.`variant` AS `variant`,
              `s`.`sold_quantity` AS `sold_quantity`,
              `g`.`gross_price` AS `gross_price_per_item`,
              ROUND((`g`.`gross_price` * `s`.`sold_quantity`),
                      2) AS `gross_price_total`,
              `pre`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`
          FROM
              ((((`fact_sales_monthly` `s`
              JOIN `dim_customer` `c` ON ((`s`.`customer_code` = `c`.`customer_code`)))
              JOIN `dim_product` `p` ON ((`s`.`product_code` = `p`.`product_code`)))
              JOIN `fact_gross_price` `g` ON (((`g`.`product_code` = `s`.`product_code`)
                  AND (`g`.`fiscal_year` = `s`.`fiscal_year`))))
              JOIN `fact_pre_invoice_deductions` `pre` ON (((`pre`.`customer_code` = `s`.`customer_code`)
                  AND (`pre`.`fiscal_year` = `s`.`fiscal_year`))))
### Post invoice discount sales
      CREATE 
          ALGORITHM = UNDEFINED 
          DEFINER = `root`@`localhost` 
          SQL SECURITY DEFINER
      VIEW `sales_postin_discount` AS
          SELECT 
              `sp`.`date` AS `date`,
              `sp`.`fiscal_year` AS `fiscal_year`,
              `sp`.`product_code` AS `product_code`,
              `sp`.`customer_code` AS `customer_code`,
              `sp`.`market` AS `market`,
              `sp`.`product` AS `product`,
              `sp`.`variant` AS `variant`,
              `sp`.`sold_quantity` AS `sold_quantity`,
              `sp`.`gross_price_per_item` AS `gross_price_per_item`,
              `sp`.`gross_price_total` AS `gross_price_total`,
              `sp`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`,
              ((1 - `sp`.`pre_invoice_discount_pct`) * `sp`.`gross_price_total`) AS `net_invoice_sales`,
              (`pos`.`discounts_pct` + `pos`.`other_deductions_pct`) AS `post_invoice_discount_pct`
          FROM
              (`sales_prein_discount` `sp`
              JOIN `fact_post_invoice_deductions` `pos` ON (((`sp`.`product_code` = `pos`.`product_code`)
                  AND (`sp`.`customer_code` = `pos`.`customer_code`)
                  AND (`sp`.`date` = `pos`.`date`))))

## Stored Procedures :
### Finding Forecast accuracy
      CREATE DEFINER=`root`@`localhost` PROCEDURE `get_forecast_accuracy`(in_fiscal_year YEAR)
      BEGIN
          with for_err_table as
          (SELECT es.customer_code,c.customer,c.market, sum(es.sold_quantity) as total_sold_qty, sum((forecast_quantity-sold_quantity)) as net_error,
      	sum((forecast_quantity-sold_quantity))*100/sum(forecast_quantity) as net_error_pct,
      	sum(abs(forecast_quantity-sold_quantity)) as abs_error,
      	sum(abs(forecast_quantity-sold_quantity))*100/sum(forecast_quantity) as abs_error_pct
      	FROM fact_act_est es join dim_customer c using(customer_code)
      	where es.fiscal_year=2021
      	group by customer_code)
      	select *, if(abs_error_pct<0,0,100-abs_error_pct) as forecast_accuracy from for_err_table
      	order by forecast_accuracy desc ;
      END
### Finding top n customers by net sales
      CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_customers_by_net_sales`(in_fiscal_year int,top_n int)
      BEGIN
      select dc.customer, round(sum(net_sales)/1000000,2) as total_net_sales
      from net_sales s join dim_customer dc on s.customer_code = dc.customer_code
      where fiscal_year = in_fiscal_year
      group by customer
      order by total_net_sales desc
      limit top_n;
      END
### Finding top n markets by net sales
      CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_markets_by_net_sales`( in_fiscal_year int, top_n int)
      BEGIN
      select market, round(sum(net_sales)/1000000,2) as total_net_sales
      from net_sales
      where fiscal_year=in_fiscal_year
      group by market
      order by total_net_sales desc
      limit top_n;
      END
### Finding top n products by net sales
      CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_products_by_net_sales`(in_fiscal_year int, top_n int)
      BEGIN
      select product, round(sum(net_sales)/1000000,2) as total_net_sales
      from net_sales
      where fiscal_year = in_fiscal_year
      group by product
      order by total_net_sales desc
      limit top_n;
      END
### Finding gross sales monthly for customer
      CREATE DEFINER=`root`@`localhost` PROCEDURE `gross_sales_monthly_for_customer`(cus_code text)
      BEGIN
      select s.date, round(sum(g.gross_price*s.sold_quantity),2) as gross_price_total from fact_sales_monthly s
      join fact_gross_price g
      on s.product_code=g.product_code and get_fiscal_year(s.date)=g.fiscal_year
      where find_in_set(s.customer_code, cus_code)>0
      group by s.date
      order by s.date;
      END
### Determining total sold quantity of a market in a given year
      CREATE DEFINER=`root`@`localhost` PROCEDURE `market_fy_total_sold_quantity`(in in_market varchar(45), in in_fy year, out market_badge varchar(45))
      BEGIN
      declare qty int default 0;
      select sum(sold_quantity) into qty
      from dim_customer c join fact_sales_monthly s on s.customer_code=c.customer_code
      where get_fiscal_year(date)=in_fy and market=in_market
      group by market;
      if qty>5000000 then set market_badge ="gold";
      else set market_badge ="silver";
      end if;
      END

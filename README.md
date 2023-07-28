# data_model

The summary shows the problems we often faced in data analysis and data migration. There are also some useful solutions for each problem.


## Data model problems
- [duplication issue](#duplication-issue)
- [integration issue](#integration-issue)
- [data gap issue](#data-gap-issue)

### duplication issue
Duplication is one of the common issues in Data handling (ETL). Duplication refers to the presence of redundant or repeated data within a data model or database. It occurs when the same information is stored in multiple places, leading to potential inconsistencies and inefficiencies in data management. There are a few scenarios for duplication

1) __Dimension Table__: Provide structured labeling information, used to group by. EX: Customer ID, Customer Name in the customer table
__case 1__: 
There's __no uniqueness defined__, so two records got the same key， and this needs to be cleaned up as This is a Data Quality issue.

![image](https://github.com/yrenn/data_model/assets/118937529/c03748fe-af09-4502-99ee-8dbb6ed87687)

Solution:
(1)Take the latest one as truth, and delete the outdated one, in this case, DateInserted=2016-01-01 will be taken, and the first one will be deleted.

(2) Add column - Version to identify them, all outdated ones will be is_outdated, the last one will be is_current 
![image](https://github.com/yrenn/data_model/assets/118937529/11fcd373-e3cb-4cb8-9610-b648129daf3a)

  __case 2__:
 there's a __Primary key__, so the two Vincent can not have the same key, this is very common in business as the user doesn't know the two   Vincent actually are the same, this is still some data Quality issue.

![image](https://github.com/yrenn/data_model/assets/118937529/88dfe71c-5b6c-4e9b-936e-c70788ff8051)

  Solution:
  (1) use
```
select * from table where name='Vincent' 
```
to find all the records for similar entities, and then base them on date of birth to identify whether they are the same.

  (2) once identified, either pick the latest one or merge them together, the key point in this scenario is to keep only one row for Vincent.

  (3) if our decision is to keep Customer key 3, then we need to go fact table to update Customer_table set Customer=3 where Customer=1 to fix the tables where we are using customer=1

```
update Customer_table set Customer=3 where Customer=1
```

2) Duplicates in __Fact table__(eg. Customer Sales), contains measures or business facts, can be count, Max/Min. 

__Case One__: ETL Error, load data twice

![image](https://github.com/yrenn/data_model/assets/118937529/98d2c44b-1f94-429b-8a39-d2864c5c3bf5)

  Solution:
  ```
  (select Customer from customer_sales where date=2016-1-1 group by customer having count(Customer)>1) a
  select distinct * into temp_table from customer_sales where customer in a and date=2016-1-1
  insert into customer_sales select* from temp_table
  ```
  
(1)filter duplicate with date 2016/1/1
(2)keep distinct records in the temp table
(3)delete duplicate in the fact table
(4)insert distinct records back into the fact table
   

3) __Duplicates in different Tables__
two sources of data got duplicated, one is from warehouse 1, and another is from warehouse 2, what happened is that the same item has been transferred from WH1 to WH2, so they reported it twice. 
 __Case 1: same primary key__
 Solution:
 (1) join the table in the same primary key
 (2) delete records in one table
 
```
 select item from Item_Inventory_WH1 join Item_Inventory_WH2 on WH1.item = WH2.item
```
 __Case 2: Different primary key__, but two table contains same item 


![image](https://github.com/yrenn/data_model/assets/118937529/778224a5-52c4-40c1-b668-2cf04d0d6d73)

Solution:
 (1) Mapping key table for both sets of items
 (2) Translate the key in the WH2 table to be the same using the mapping key table 
 (3) Now we have two tables, then the same steps as Case One above

```
select distinct.WH1.item, WH2.item into keymap_table from Item_Inventory_WH1 	join Item_Inventory_WH2 on WH1.itemName=WH2.itemName
select Warehouse, item, itemName, Qty into Ts_item_inventory_WH2 from 	Item_inventory_WH2 join keymap_table on WH2.item=keymap_table.item
```

![image](https://github.com/yrenn/data_model/assets/118937529/e1bf5e3a-ee74-479b-89ba-14a4e69768aa)

### integration issue

1. Diverse Data Sources or different databases
Data from different sources may be stored in different file formats (such as CSV, Excel, or JSON) or different database systems (such as MySQL, Oracle, or MongoDB). In order to integrate this data, it may be necessary to convert it to a common format or to extract and transform it in a way that allows it to be combined with other data sources.

2. Low-Quality Information
  Incomplete data: Data may be missing important information or have incomplete records, which can make it difficult to use or analyze.
  Incorrect data: Data may contain errors or inaccuracies, which can lead to incorrect conclusions or decisions based on the data.
  Outdated data: Data may be outdated or no longer relevant, which can also impact its usefulness.
  Poorly formatted data: Data may be stored in a way that makes it difficult to integrate or analyze, such as in different formats or structures.

### data gap issue

1. Missing data
Your organization does not gather the full scope of available information.
2. Data disparity
The disparity between the importance that management allocates to a metric or indicator.
3. Misinterpretation of data
If the individual analyzing the data does not have the right knowledge and skill to interpret or clean the data. This results in drawing an incorrect conclusion and possibly making a harmful decision.



### Normal data model problems would be faced
1) Inaccurate Customer and Contact Data （missing values
2) Big Data Makes it Hard to Focus (wrong values, collect various data)
Big data captured generally fall into three categories: unstructured (text, videos, audio), semi-structured data (email, reports, spreadsheets), and structured data (sensor data, machine data, financial models, algorithms, etc.).
3) Compliance Issues
Consolidating privacy and security compliance management as part of an overall data governance initiative gives a business a significant advantage.
4) Duplicate or Obsolete Issues
 If a company commonly has multiple records for the same customer in a database or data warehouse, duplicate identification tools can help match and link records based on individual standards, allowing for greater accuracy over time.
5) Reconciling Data Quality Issues During a Merger (information in two ERP systems may be different)
Most data quality issues can be significantly reduced through careful planning during the design phase and data integration effort. At the start of a merger, for example, a company should take time to regroup and create standards, guidelines, and procedures for corporate data processes.
6) Inaccurate Transaction Data (duplicate data or records)
data marts are used as smaller data storage facilities to provide reporting and analytical capabilities for specific business processes.
7) Operational Productivity Roadblocks
 Productivity decreases in instances of lost products and time due to reprocessing an order shipped to an outdated address, or an increase in time spent flagging and fixing invalid records.
 solution: Using a new platform such as Hubspot is deployed to track inbound marketing activities, the enterprise data quality initiative should be scalable to address those new requirements.
8) Growth and Expansion Impacts
A manufacturer, for instance, may have limited internal sales data, requiring third-party data from retailers to get a full picture of their end customer. Adding new facilities and enriching internal base data with third-party sources. 
9) Select Data governance leads
It’s important to assign responsibilities and formal procedures for each data governance lead and everyone that handles data. During this phase, it’s also critical that IT has a firm grip on data quality expectations so they can build the system to support the right data types.
11) Using Flawed Data()
it’s important to see the big picture and set realistic goals and an iterative approach to any data quality initiative and take a strong data quality program

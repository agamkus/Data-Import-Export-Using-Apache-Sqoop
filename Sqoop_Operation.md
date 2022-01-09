# Data-Import-Export-Using-Apache-Sqoop

## Problem Statement:
Apache Sqoop, which is short for ‘SQL to Hadoop’, is used for ingesting relational data. Essentially, it connects an RDBMS, such as MySQL and Oracle, to the HDFS and provides efficient, bidirectional data transfer between them in parallel, as shown in the image given below.
Data can be imported directly into the HDFS, or to HBase or Hive tables, as per requirement. When you use Sqoop to transfer data, the data set being transferred is split into multiple partitions and a map-only job is launched. Individual mappers are then responsible for transferring each slice/partition of the data set. The metadata of the database is used for handling each data record in a type-safe manner.
Our objective here is to import/export data between MySQL and HDFS. We’ll work on the 3 datasets and import specific records into the different directories of HDFS. We are required to perform below operations during the activity:
- Export data from HDFS to MySQL
- Import Data from MySQL to HDFS
- Import data from all the tables
- Handle NULL Values
- Handle Number of Mappers
- Import in different File Formats
- Apply compression on imported data
- Import Specific Rows
- Use SQL Queries in Import
- Incremental Import
- Create and Execute Sqoop Jobs
- Sqoop Tuning

## Data Sets:
We’ll be going through the following three data sets for our case study on Apache Sqoop:
- Flights data set
- Retail data set
- Employee data set

## Flights Data Set:
The Flights data set has 3 columns, Destination, Origin and Count which contain following information:
- The **Destination** column consists of destination country names where flights will land.
- The **Origin** column consists of origin country names from where flights will take off.
- The **Count** column contains the number of flights between the specified destination and origin countries.

## Retail Data Set:
Companies maintain an online Retail data set to keep track of all the goods being sold to their customers across different countries. The Retail data set contains following **8** columns:
- The **Invoice Number** column is used for tracking invoices and all the orders associated with them.
- The **Stock Code** column contains the commodity code for a particular item.
- The **Description** column contains the description of a commodity.
- The **Quantity** column contains the quantity of a commodity available in stock or in the warehouse for sale.
- The **Invoice Date** column consists of the date on which the invoice was generated.
- The Unit **Price** column contains the price of a single unit of a particular commodity.
- The **Customer ID** column consists of the ID of a customer who purchased a particular item.
- The **Country** column contains the name of the country where a particular item is to be sold.

## Employee Data Set:
The **Employee** data set is used for keeping track of their employees. It contains following 4 columns:
- The **Employee ID** column is the primary key of this table, and it is used for keeping track of every employee.
- The **First Name** column contains the first names of all the employees.
- The **Designation** column consists of the designations of all the employees.
- The **Salary** column contains the salary paid by the company to a particular employee.

## Execution Steps:
1. Start the AWS EC2 instance by logging in to AWS Management Console.
2. Download Flights and Retail data set using below commands:

*wget -P /root/ https://s3.amazonaws.com/sqoop.dse.2020/flights_data.txt<br>
wget -P /root/ https://s3.amazonaws.com/sqoop.dse.2020/online_data.txt*

![image](https://user-images.githubusercontent.com/56078504/148688472-c338798f-a30f-4428-9e65-6461dd878dcb.png)

![image](https://user-images.githubusercontent.com/56078504/148688480-ca79221f-7391-47c5-9b40-8cb8b78789d8.png)

Once the data files are downloaded to the local AWS EC2 instance, they need to be moved from /root to some HDFS location for further analysis.

3.	Run the following commands to move the data from the local root directory to the HDFS location, so it could be further exported to MySQL:

*hadoop fs -put /root/flights_data.txt /user/root/flights_data<br>
hadoop fs -put /root/online_data.txt /user/root/online_data*

4.	After this, verify the data in the HDFS location by using the following commands:

hadoop fs -ls /user/root/flights_data<br>
hadoop fs -ls /user/root/online_data

![image](https://user-images.githubusercontent.com/56078504/148688934-ae9bd66b-7698-4a76-80ed-edb3e0427bf4.png)

5.	Next, login to MySQL using below commands:

*mysql -u root -p<br>
Enter Password: *** *

![image](https://user-images.githubusercontent.com/56078504/148688965-76eb1c4c-80fe-4aed-a0e3-f8560eb60653.png)

6.	Run the following commands to create a database **recipes_database** and tables **recipes**, **ingredients** and **recipe_ingredients**:

*CREATE DATABASE recipes_database;<br>
USE recipes_database;*

*CREATE TABLE recipes (recipe_id INT NOT NULL, recipe_name VARCHAR(30) NOT NULL, PRIMARY KEY (recipe_id), UNIQUE (recipe_name));
INSERT INTO recipes (recipe_id, recipe_name) 
VALUES (1, "Tacos"), (2, "Tomato Soup" ), (3, "Grilled Cheese" );*

*CREATE TABLE ingredients ( ingredient_id INT NOT NULL, ingredient_name VARCHAR(30) NOT NULL, ingredient_price INT NOT NULL, PRIMARY KEY (ingredient_id), UNIQUE (ingredient_name));
INSERT INTO ingredients (ingredient_id, ingredient_name, ingredient_price) VALUES (1, "Beef" , 5), (2, "Lettuce" , 1), (3, "Tomatoes" , 2), (4, "Taco Shell" , 2), (5, "Cheese" , 3), (6, "Milk" , 1), (7, "Bread" , 2);*

*CREATE TABLE recipe_ingredients ( recipe_id int NOT NULL, ingredient_id INT NOT NULL, amount INT NOT NULL, PRIMARY KEY (recipe_id,ingredient_id));
INSERT INTO recipe_ingredients (recipe_id, ingredient_id, amount)
VALUES (1,1,1), (1,2,2), (1,3,2), (1,4,3), (1,5,1), (2,3,2), (2,6,1), (3,5,1), (3,7,2);*

7.	Now, we can run the following commands (one-by-one) to verify whether the data has been loaded successfully into the tables:

*show tables;
select * from ingredients;<br>
select * from recipe_ingredients;<br>
select * from recipes;<br>*

![image](https://user-images.githubusercontent.com/56078504/148691449-c406d089-0fb4-4b3e-90dd-5d3865cfd5b6.png)

![image](https://user-images.githubusercontent.com/56078504/148691457-f5e21423-6676-4f52-a2f6-1db66cb59db9.png)

![image](https://user-images.githubusercontent.com/56078504/148691460-0effb59d-e211-45d1-98d1-f23e502bf056.png)

![image](https://user-images.githubusercontent.com/56078504/148691468-049fc4b9-6cf4-47a7-9424-bccb91f90884.png)

8.	Next, we need to create a database **test** and 3 tables **employee**, **retailinfo** and **flights_info**, which we would use for the rest of the sqoop import and export commands:

*create database test ;<br>
use test ;*

*create table employee (id INT, first_name VARCHAR(150), designation VARCHAR(150), salary INT, PRIMARY KEY (id));*

*insert into employee values (100, 'Harbhajan' , 'Software Engineer' ,5000);*<br>
*insert into employee values (101, 'Yuvraj' , 'Senior Software Engineer' ,7000);*<br>
*insert into employee values (102, 'MS Dhoni' , 'Manager' ,10000);*<br>
*insert into employee values (103, 'Sachin Tendulkar' , 'Senior Manager' ,11000);*<br>
*insert into employee values (104, 'Virat Kohli' ,null, 7000);*

Run below commands to create below 2 empty tables **retailinfo** and **flights_info** for sqoop export operation:

*create table test.retailinfo(invoiceno varchar(150),stockcode varchar(150),description varchar(150),quantity int, invoicedate varchar(150),unitprice double,customerid int,country varchar(150));*

*Create table test.flights_info (destination VARCHAR(150), origin VARCHAR(150), count INT);*

Check all the tables created:

![image](https://user-images.githubusercontent.com/56078504/148691853-dd7dd913-2a1f-4f15-bba4-06772313b42a.png)

*select * from employee;*

![image](https://user-images.githubusercontent.com/56078504/148691876-9ff5733d-dcb2-49b3-b5a4-ebf15f3ae255.png)

## Segment 1 - Sqoop Export:
1.	In order to load the data from a file in the HDFS to an RDBMS, we can use the **Sqoop export** command. Run the following commands to export **retailinfo** data:

*sqoop export --connect jdbc:mysql://localhost/test \
--table retailinfo \
--username root --password 123 \
--export-dir /user/root/online_data \
--fields-terminated-by ',' --lines-terminated-by '\n'*

![image](https://user-images.githubusercontent.com/56078504/148691932-2d4b6d8a-6db2-44e5-994d-a117aadb9201.png)

2.	Similarly, we will export **flight data**, using the below Sqoop export command:

*sqoop export --connect jdbc:mysql://localhost/test \
--table flights_info \
--username root --password 123 \
--export-dir /user/root/flights_data \
--fields-terminated-by ',' --lines-terminated-by '\n'*

![image](https://user-images.githubusercontent.com/56078504/148691997-19378826-07ed-4fab-ac36-5ec213b81546.png)

3.	After exporting the data using Sqoop, data verification can be done by looking at the count of data in MySQL, as shown below:

![image](https://user-images.githubusercontent.com/56078504/148692060-6c1c3efe-ea63-462b-ae50-fb9c433d0178.png)

## Segment 2 - Sqoop Import:

This is a basic Sqoop import command, which is used to pull data from MySQL and insert it into a specific target directory in the HDFS.
1.	Before running the Sqoop command, we will check how many records are present in the MySQL table flights_info.

![image](https://user-images.githubusercontent.com/56078504/148692880-7604a93e-6829-47c5-907b-015a579b26e2.png)

So, we found that total **253 records* are there in flights_info table.

```diff
- **Note:** Before running the Sqoop import command, make sure the target directory is not already created; otherwise, the Sqoop import command will throw an error.
```

```diff 
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```












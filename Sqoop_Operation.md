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
Enter Password: \****

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
1.	Before running the Sqoop command, we will check how many records are present in the MySQL table **flights_info**.

![image](https://user-images.githubusercontent.com/56078504/148692880-7604a93e-6829-47c5-907b-015a579b26e2.png)

So, we found that total **253 records** are there in **flights_info** table.

***Note: Before running the Sqoop import command, make sure the target directory is not already created; otherwise, the Sqoop import command will throw an error.***

2.	After getting a count of the records in the MySQL table, we have to verify the number of records imported using the Sqoop command. So, run the Sqoop import command and verify.

*hadoop fs -rm -r /user/root/flights_basic_command*

*sqoop import \
--connect jdbc:mysql://localhost/test \
--table flights_info \
--username root --password 123 \
--target-dir /user/root/flights_basic_command \
-m 1*

![image](https://user-images.githubusercontent.com/56078504/148720003-ebcdbea8-94fe-46e5-aa0b-e4561869cdf7.png)

![image](https://user-images.githubusercontent.com/56078504/148720030-ba981649-8e08-452d-a972-3a797d2f9017.png)

We see that total 253 records were imported. After importing the data from Sqoop, if the count of the records matches the count of the records in MySQL, then it means that the import was successful.

3.	Run the below command to check the HDFS target directory where the data was imported:

*hadoop fs -ls /user/root/flights_basic_command*

![image](https://user-images.githubusercontent.com/56078504/148720099-a0e50241-9248-47b2-b3e0-cb6a530857fb.png)

## Segment 3 – Importing all Tables using sqoop:
Sqoop has a command for importing all MySQL tables belonging to a specific database in a single execution itself.

Before executing the command, make sure to count the number of tables in MySQL, because the same number of target directories would be created in the home Hadoop location, i.e., ‘hadoop fs -ls /user/root/’

***Note: A point to be noted here is that all the tables in MySQL should have a primary key defined; otherwise, the command will throw an error.***

1.	Run following commands to import records of all the tables from the database **recipes_database**. Before that let’s remove the existing directories, if any:

*hadoop fs -rm -r /user/root/recipes
hadoop fs -rm -r /user/root/recipe_ingredients
hadoop fs -rm -r /user/root/ingredients*

*sqoop import-all-tables \
--connect jdbc:mysql://localhost/recipes_database \
--username root \
--password \****

![image](https://user-images.githubusercontent.com/56078504/148720837-627040cf-5c98-4d52-a206-6cf166f999fa.png)

2.	After executing the import-all-tables command, verify the data in the Hadoop location. We should have the same number of directories corresponding to the number of tables in MySQL. Run the below command to check the HDFS target directory where the data was imported:

*hadoop fs -ls /user/root/ingredients
hadoop fs -ls /user/root/recipes
hadoop fs -ls /user/root/recipe_ingredients*

![image](https://user-images.githubusercontent.com/56078504/148720860-5ad4653a-4f96-43e3-9b96-c9fa48e0e0d7.png)

## Segment 4 – Handling NULL Values
1.	Sqoop has commands **--null-string** and **--null-non-string** that provide an option to handle NULL values as well. If we do not put this clause, then Sqoop will import NULL values as a string, with the values of these strings written as ‘null’. Hence, to make sure NULL values are imported as they are in MySQL, select the following clause while importing data using Sqoop:

*sqoop import \
--connect jdbc:mysql://localhost/test \
--table employee \
--username root --password 123 \
--null-string '\\N' --null-non-string '\\N' \
--target-dir /user/root/employee_null_command \
-m 1*

![image](https://user-images.githubusercontent.com/56078504/148720886-93eb21cf-6178-4f09-be90-cd00646bc26e.png)

Sqoop allows us to import data in the file formats that do not generally support the NULL value. Hence, you need to encode the missing value or the NULL value in the data. Sqoop encodes the missing value to a string constant 'null' (in lowercase).

However, there is a problem with this approach. If your data itself contains NULL as a string/regular value, rather than a missing value, then this type of encoding does not help. Also, further processing steps may require a different substitution for missing values.

For the missing values in text-based columns, we use the ‘--null-string’ parameter, and for other columns such as columns where only numbers are present, we use the ‘--null-non-string’ parameter.

2.	Run the below command to check the HDFS target directory where the data was imported:

*hadoop fs -ls /user/root/employee_null_command
hadoop fs -cat /user/root/employee_null_command/part-m-00000*

![image](https://user-images.githubusercontent.com/56078504/148720915-e1f0261a-aeac-4403-9c2c-68bfc9c91b40.png)

## Segment 5 – Handling Mappers for a Sqoop job
Sqoop import command has a parameter that provides an option to handle the number of mappers as well. You can increase and decrease the number of mappers as per your cluster memory.
The number of part files getting created is equal to the number of mappers used in the Sqoop command.

1.	Run the following commands to import data from **employee** table with **4 mappers**:

*sqoop import \
--connect jdbc:mysql://localhost/test \
--table employee \
--username root --password 123 \
--target-dir /user/root/employee_mapper_command \
--split-by id -m 4*

![image](https://user-images.githubusercontent.com/56078504/148720963-30865250-5970-48b5-9e08-75f9303a9b6e.png)

We can see that now 4 mappers are triggered. Below is the screenshot:

![image](https://user-images.githubusercontent.com/56078504/148720974-d08ec35a-6b7f-4a12-9670-0c71089cfa43.png)

***Note: While specifying the number of mappers, make sure to provide the split id column, which is usually the primary key of that table.***

2.	Run the below command to check the HDFS target directory where the data was imported:

*hadoop fs -ls /user/root/employee_mapper_command*

![image](https://user-images.githubusercontent.com/56078504/148721018-5bfea710-8963-460b-ad47-256790efa390.png)

## Segment 6 – Importing Data in Different File Formats
Sqoop can be used to import data in different file formats. It does not matter how the data is stored in MySQL; when you provide the option in the Sqoop command to store the data in a specific file format, then the Hadoop MapReduce job running at the backend takes care of it automatically.

1.	Run the following commands to import data from **flights_info** table in **parquet** file format:

*sqoop import \
--connect jdbc:mysql://localhost/test \
--table flights_info \
--username root --password 123 \
--target-dir /user/root/flights_parquet_command \
-m 1 \
--as-parquetfile*

![image](https://user-images.githubusercontent.com/56078504/148721057-f506f0ff-d051-463e-a8af-9f540596752b.png)

2.	Run the following commands to import data from **flights_info** table in **sequence** file format:

*sqoop import --connect jdbc:mysql://localhost/test \
--table flights_info \
--username root --password 123 \
--target-dir /user/root/flights_sequence_command \
-m 1 --as-sequencefile*

![image](https://user-images.githubusercontent.com/56078504/148721078-c32ad464-e044-4c2a-ad65-2d474347715f.png)

3.	Check the target location using below commands:

*hadoop fs -ls /user/root/flights_parquet_command
hadoop fs -ls /user/root/flights_sequence_command*

![image](https://user-images.githubusercontent.com/56078504/148721405-c6949713-2249-4862-a902-f38cf14c1839.png)

![image](https://user-images.githubusercontent.com/56078504/148721410-3bed6612-dffe-4fcb-8b0d-ef8215b8539c.png)

## Segment 7 – Compression using Sqoop
We can compress the data while using the sqoop import command.

1.	Run the below command with Compression Codec enabled as **SnappyCodec**:

*sqoop import --connect jdbc:mysql://localhost/test \
--table flights_info \
--username root --password 123 \
--target-dir /user/root/flights_compress_command \
-m 1 --as-sequencefile \
--compression-codec org.apache.hadoop.io.compress.SnappyCodec*

![image](https://user-images.githubusercontent.com/56078504/148721438-0cff47a5-85c8-4394-9756-659ab2751ef4.png)

2.	Navigate to the target directories and note down the file sizes with and without compression enabled.

*hadoop fs -ls /user/root/flights_sequence_command
hadoop fs -ls /user/root/flights_compress_command*

![image](https://user-images.githubusercontent.com/56078504/148721461-532db50a-3aa6-453f-a152-27e6fb63a8ac.png)

![image](https://user-images.githubusercontent.com/56078504/148721472-5b0ad443-bb9a-4335-ac6d-e406f14bf800.png)

## Segment 8 - Importing Specific Rows in Sqoop
Sqoop provides the option to import specific rows as well, based on some **where** conditions similar to SQL.

1.	First, run the where clause in MySQL to determine the number of rows that are getting abstracted out of that query, as shown below:

*select count(*) from test.flights_info where destination<>origin;*

 ![image](https://user-images.githubusercontent.com/56078504/148721509-49e9eeb0-2b51-4123-ad9d-950557308889.png)


2.	Run the Sqoop import command with the where clause as shown below to import records that satisfy the where condition:

*sqoop import --connect jdbc:mysql://localhost:3306/test \
--table flights_info \
--username root --password 123 \
--target-dir /user/root/flights_where_command \
--where "destination<>origin" \
-m 1*

![image](https://user-images.githubusercontent.com/56078504/148721537-17e89dbb-3601-4447-b89c-e3366cc29d53.png)

![image](https://user-images.githubusercontent.com/56078504/148721546-60708cc3-0418-4b56-84e1-c54d60c615e5.png)

So as per the above screenshot, we can see that total **252 records** got imported where origin is not equal to destination.

## Segment 9 - SQL Queries in Sqoop Import
Sqoop provides the option to import data satisfying SQL query conditions. The **--query** parameter holds the query and the **--split-by** parameter indicates the column that is to be used for splitting the data into parallel tasks. By default, this column is the **primary key** of the main table.

Run below commands to import all data of **employee** table using SQL query:

*sqoop import --connect jdbc:mysql://localhost/test \
--username root --password 123 \
--query 'select * from/test.employee where $CONDITIONS' \
--split-by id \
--target-dir /user/root/flights_query_command*

 ![image](https://user-images.githubusercontent.com/56078504/148721714-1815819c-352a-4bcb-8dd0-24f73d0638b0.png)

**'$CONDITIONS'** is a placeholder that would be substituted automatically by Sqoop to indicate which slice of data is to be transferred by each task. This type of import command is also known as free-form query import. Sqoop does not use the database catalogue to fetch the metadata while performing free-form query imports.

## Segment 10 - Incremental Import in Sqoop
In order to perform incremental import in Sqoop, there should be a column in the corresponding table which should be either a primary key or a column on which incremental append conditions could be applied.

Through this command, Sqoop provides us an option to import newly appended records only, based on the last value that is written.

1.	First of all, run below commands to append 2-3 records in MySQL using an insert clause:

*insert into/test.employee values (105, 'Hitesh' , 'software engineer' ,2000);
insert into/test.employee values (106, 'Rahul' , 'senior software engineer' ,3000);*

![image](https://user-images.githubusercontent.com/56078504/148721965-c5a48df0-5c6b-4a32-87a5-6b075d1b9e6e.png)

![image](https://user-images.githubusercontent.com/56078504/148721967-3be154b2-1fde-4b0e-b773-dc82daa78f21.png)
 

2.	Now, run the sqoop import command to fetch only those records from employee table which are added after the ID value 104 as shown below:

*sqoop import --connect jdbc:mysql://localhost/test \
--table employee --username root --password 123 \
--target-dir /user/root/employee_incremental_command \
--incremental append --check-column id \
--last-value 104 \
-m 1* 

![image](https://user-images.githubusercontent.com/56078504/148721990-52ff4aaf-1b08-4ab4-a2a0-3555b2cf663d.png)

![image](https://user-images.githubusercontent.com/56078504/148721995-7336882a-4b7c-484f-a6f8-050fb8e40bd2.png)

So, as per the above screenshot, 2 newly created records have been imported by sqoop operation.

3.	Verify the newly fetched records in the target directory.

![image](https://user-images.githubusercontent.com/56078504/148722013-d80a7927-01ab-4fdd-967b-b385c4511414.png)

## Segment 11 - Sqoop Job
Sqoop provides an option to save the import and export commands so that they can be executed repeatedly.

1.	Run below commands to create an import job **flightsimport** which imports data from the table **flights_info**:

*sqoop job --create flightsimport -- import \
--connect jdbc:mysql://localhost:3306/test \
--table flights_info \
--username root --password 123 \
--target-dir /user/root/flights_job_command \
-m 1*

![image](https://user-images.githubusercontent.com/56078504/148722166-bb427e56-6dfc-4346-89e7-aa1994b381ea.png)

2.	We can check whether the job was created or not by running the list option with the sqoop job command, as shown below:

*sqoop job –list*

![image](https://user-images.githubusercontent.com/56078504/148722191-496dee3c-efbe-417f-a2cf-6e62092e9be3.png)

3.	The created job will not work until we execute it. Now, we can execute the sqoop job by following command. Before executing the job, keep in mind that the target directory should not exist at all.

*sqoop job --exec flightsimport*

![image](https://user-images.githubusercontent.com/56078504/148722208-7e79095d-9018-4e48-8b6b-804348909816.png)

4.	Verify the data by checking the target directory once.

*hadoop fs -ls /user/root/flights_job_command*

![image](https://user-images.githubusercontent.com/56078504/148722230-2d144aa8-60e8-4cf8-833e-f8f658c1e950.png)
 
We can delete a sqoop job using below command:

*sqoop job --delete flightsimport*

Once you have a Sqoop job created, it can then be scheduled with workflow management tools such as **Apache Oozie** or **Airflow** and can be triggered based upon an event.

## Segment 12 - Tuning Sqoop
While working on a project, especially in the production environment, password is not passed as a parameter along with the Sqoop command. Rather, it is stored in a text file, with only read permissions to the User ID through which the Sqoop command is running in production.
***Note: In order to store passwords in a text file, keep in mind that there should not be any ‘\n’ character after the password in the text file; otherwise, the Sqoop import command will keep on failing and we will not be able to debug the issue properly.***

1.	Run the below command to create a file **pass.txt**:

*echo -n "123" > pass.txt*

 ![image](https://user-images.githubusercontent.com/56078504/148722371-607cb59d-fef2-4797-8ba2-af8f0f1774e2.png)

2.	After storing the password in a local text file, move that file to the Hadoop location and assign **chmod 400** permissions to it. 400 denotes read only permission to the owner.

*hadoop fs -put /root/pass.txt /user/root/pass.txt
hadoop fs -chmod 400 /user/root/pass.txt*

![image](https://user-images.githubusercontent.com/56078504/148722402-11537d86-9d65-4f4c-bcda-695b8cacc842.png) 

3.	Now, run the sqoop import command with parameter **--password-file** as shown below:

*sqoop import \
--connect jdbc:mysql://localhost/test \
--table flights_info \
--username root --password-file /user/root/pass.txt \
--target-dir /user/root/flights \
-m 1*

 ![image](https://user-images.githubusercontent.com/56078504/148722430-b16395ea-e556-4522-a48c-4f21e3b190bd.png)


4.	Use the following command for verifying the data in the target directory:

*hadoop fs -ls /user/root/flights*

 ![image](https://user-images.githubusercontent.com/56078504/148722442-384b04af-ffd8-4df6-9b10-2e353da45a85.png)


## Conclusion:
As per the above case study, we found that Sqoop can perform various import/export operation between RDBMS and HDFS as per our requirements. To import huge records, we can control the number of mappers using **-m** parameter in the sqoop commands. However, increasing the number of mappers does not necessarily reduce the processing time. By default, Sqoop launches 4 mappers.

The database may get overwhelmed by a large number of mappers and lose time in context switching between these tasks rather than transferring the data. Below you can see the general trend of how the processing speed of the Sqoop import statement increases as the number of Mappers for the Sqoop job is increased.
 
![image](https://user-images.githubusercontent.com/56078504/148722461-3d3fef9a-ae20-40e1-b0ee-467685513cd6.png)

The best method to determine the optimal number of mappers is trial and error. You can define the number of mappers at a starting value, and ten increase it and test it until you achieve no further improvement.

































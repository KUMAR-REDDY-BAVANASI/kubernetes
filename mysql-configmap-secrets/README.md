connect to mysql database
=========================
mysql -u kumar_user -p 


mysql commands
===============
```
show databases;                		==> List out the databases
use <database-name>;				==> Connect and switch to your database
show tables;                    	==> List out the tables in database
select * from <tablename>;	     	==> Fetch data from tablename in database
DROP TABLE <tablename>;             ==> Delete the table from database
DROP DATABASE <database-name>;   	==> Delete the database
exit								==> Exit from database terminal
```

reate Database
---------------
CREATE DATABASE petclinic;

Switch to Database
------------------
use petclinic;

Create Table
------------
```
CREATE TABLE petclinic (
   id SERIAL PRIMARY KEY,
   firstName VARCHAR(255) NOT NULL,
   lastName VARCHAR(255) NOT NULL
);
```

Fetch the Database Tables
-------------------------
show tables;

Insert the data into table
--------------------------
INSERT INTO petclinic (firstName, lastName) VALUES('Kumar REDDY','BAVANASI');

Fetch the data from table
-------------------------
select * from petclinic;

Delete the table
----------------
DROP TABLE petclinic;

Delete the database
----------------
DROP DATABASE petclinic;
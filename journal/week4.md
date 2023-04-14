# Week 4 — Postgres and RDS
## Setup Postgress RDS on AWS using Console
```sh
aws rds create-db-instance \
  --db-instance-identifier cruddur-db-instance \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version  14.6 \
  --master-username root \
  --master-user-password goodDatabasePassword1CruddurApplication \
  --allocated-storage 20 \
  --availability-zone us-east-1d \
  --backup-retention-period 0 \
  --port 5432 \
  --no-multi-az \
  --db-name cruddur \
  --storage-type gp2 \
  --publicly-accessible \
  --storage-encrypted \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --no-deletion-protection
```
**NB** When setting up an db instance - **important** to set the timezone and character set type to ensure no confusion with the imported database

**NB:** Error occured above due to lack of a default VPC Subnet </br>
**Solution**</br>
1. Create a default VPC using the CLI command
```sh
aws ec2 create-default-vpc
```
**Insert image**</br>
*Repaste the RDS command this time it should work*

https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html#default-subnet

**Insert image**

To connect to psql via the psql client cli tool remember to use the host flag to specific localhost.

```sql
psql -Upostgres --host localhost
# --host localhost needs to be set due to docker setup
```
Common PSQL Commands

```sql
\x on -- expanded display when looking at data
\q -- Quit PSQL
\l -- List all databases
\c database_name -- Connect to a specific database
\dt -- List all tables in the current database
\d table_name -- Describe a specific table
\du -- List all users and their roles
\dn -- List all schemas in the current database
CREATE DATABASE database_name; -- Create a new database
DROP DATABASE database_name; -- Delete a database
CREATE TABLE table_name (column1 datatype1, column2 datatype2, ...); -- Create a new table
DROP TABLE table_name; -- Delete a table
SELECT column1, column2, ... FROM table_name WHERE condition; -- Select data from a table
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...); -- Insert data into a table
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition; -- Update data in a table
DELETE FROM table_name WHERE condition; -- Delete data from a table
```
https://www.postgresql.org/docs/current/app-createdb.html

We can create the database within the PSQL client
```sql
CREATE database cruddur;
```


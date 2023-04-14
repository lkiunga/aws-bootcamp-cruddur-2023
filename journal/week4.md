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
Create a new db folder on the `backend` directory. Add a file `schema.sql`.

Add the following to the schema ID

  1. Add UUID extension  - used to obsecure the user unique identify which enables us to hide the number of users in our application especially for       competitor marketing purpose.


```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

```
To run the `schema.sql` run the following code on the backend directory folder
```sql
psql cruddur < db/schema.sql -h localhost -U postgres
```
To avoid typing in the password- setup the CONNECTION_URL variable for the local psql client
```sh
export CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"
```
To check whether it is working connect like this
```sh
psql $CONNECTION_URL 
```
#### Persist the ENV variable

Using the command line: gp env - The gp CLI prints and modifies the persistent environment variables associated with your user for the current repository.
```sh
gp env CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"
```
To avoid typing in the password- setup the CONNECTION_URL variable for the Production env **AWS RDS Instance**
  export PROD_CONNECTION_URL="postgresql://cruddurroot:input_your_own_password@cruddur-db-instance.c5at744wtgv6.us-east-1.rds.amazonaws.com:5432/cruddur"

  gp env PROD_CONNECTION_URL="postgresql://cruddurroot:input_your_own_password@cruddur-db-instance.c5at744wtgv6.us-east-1.rds.amazonaws.com:5432/cruddur"

## BASH Scripting
Create a `bin` directory and add create the following files ` db-create, db-drop, db-schema-load`

We will be creating scripts to create our database, drop the database and build the database schema.

#### To make files executable 
   1. add the bash link file 

      Type the following on the shell `whereis bash` and copy the bash file and add it to the first line of the files inside the bin folder 
      ```sh
       #! /usr/bin/bash
      ```
  2. Change the file permission values

      By executing the following command 
      ```sh
      chmod U+x db-create, db-drop, db-schema-load
      ```
      this turns the files into executable files
  
#### Upload the following to the db files created
db-drop
```sh
#! /usr/bin/bash

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "DROP DATABASE cruddur;"
```
db-create
```sh
#! /usr/bin/bash

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "create database cruddur;"
```
db-schema-load
```sh
#echo "== db-schema-load"
#Makes the 
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-schema-load"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"
schema_path="$(realpath .)/db/schema.sql"

echo $schema_path

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL cruddur < $schema_path
```
**insert image

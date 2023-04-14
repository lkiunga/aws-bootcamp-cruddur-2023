# Week 4 — Postgres and RDS
## Setup Postgress RDS on AWS using Console
```
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
**NB:** Error occured above due to lack of a default VPC Subnet </br>
**Solution**</br>
1. Create a default VPC using the CLI command
```
aws ec2 create-default-vpc
```
**Insert image**</br>
*Repaste the RDS command this time it should work*

https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html#default-subnet

**Insert image**

# MySQL to S3 Data Migration using AWS DMS

This repository contains instructions and configuration steps for migrating data from a MySQL database to Amazon S3 using AWS Database Migration Service (DMS).

## Table of Contents

- [Roles and S3 Buckets](#roles-and-s3-buckets)
- [EC2 Instance Creation](#ec2-instance-creation)
- [Connecting with Terminal](#connecting-with-terminal)
- [RDS MySQL Database Creation](#rds-mysql-database-creation)
- [Create Replication Instance](#create-replication-instance)
- [Create Source Endpoint](#create-source-endpoint)
- [Create Target Endpoint](#create-target-endpoint)
- [Create Database Migration Task](#dms---create-database-migration-task)
- [Check S3 for CSV Files](#check-s3-for-csv-files)
- [Delete Resources](#delete-resources)

## Roles and S3 Buckets

### Create DMS Roles

1. Navigate to the **IAM dashboard** -> Click on **Roles** -> Click the **Create role** button.
   
2. Choose **AWS service** as the type of trusted entity.

3. For the use case, select **DMS" (Database Migration Service)**.

3. Attach the following policies to the role:
   - AmazonS3FullAccess: Grants full access to Amazon S3.
   - AmazonRDSFullAccess: Grants full access to Amazon RDS.

5. Provide a name for the role **MySqltos3** and a description.

6. Click **Create role** and copy the ARN for later use.

### Create S3 Bucket
1. Navigate to the **S3 dashboard** -> Click **Create bucket.**
2. Enter the following details:
   - Bucket name: **mydmstos3**
   - Region: **us-east-1** (or your preferred region).
3. Keep **all default settings** and **create the bucket**.

## EC2 Instance Creation

1. Navigate to the EC2 dashboard-> Click **Launch Instance**.-> Choose the **Amazon Linux 2 AMI** in the Default VPC.

2. Add tags with key: **Name** and value: **mysql-client**

3. Configure the security group named **SG-mysql** to open ports 22 (SSH) and 3306 (MySQL/Aurora).

4.  **Launch the instance and connect with SSH**

## Connecting with Terminal

1. SSH into your EC2 instance.

2. Update the package manager:
   ```bash
   sudo yum update -y

3. Install the MySQL client:
   ```bash
   sudo yum install mysql -y

4. Check the MySQL or MariaDB version:
   ```bash
   mysql --version

## RDS MySQL Database Creation
we will create an RDS (Relational Database Service) MySQL database for use in our migration.

1. Navigate to the RDS dashboard -> Click on **Databases** -> Click the **Create database** button.

2. Choose the **Easy create** method for database creation.
   
4. Configure the following settings:
   - **Engine type**: MySQL
   - **DB instance size**: Free tier (or choose an appropriate size for your needs)
   - **DB instance identifier**: `db-mysql-s3`
   - **Master username**: `admin`
   - **Master password**: `admin1234!` (Choose a strong and secure password.)

6. Click the **Create database** button to initiate the database creation process. Wait for the database to become available.

### Step 2: Modify RDS Security Group
1. In the RDS dashboard, select the database you just created **db-mysql-s3** -> Navigate to the **Connectivity & security** tab.

3. In the **Security** section, click on **VPC security groups.**

4. You will see the **default** security group. Click on **default** to open it.

5. In the **Inbound rules** tab, click **Edit inbound rules.**

6. Add a new inbound rule with the following settings:
   - **Type**: MySQL/Aurora
   - **Protocol**: TCP
   - **Port range**: 3306
   - **Source**: Select the security group used by your EC2 instance **sg-client**.

7. Click **Save rules** to apply the changes.

Now, your RDS MySQL database is accessible from the EC2 instance you created earlier using the security group **sg-client**. You can connect to this database from your EC2 instance and perform data migration tasks.

## RDS MySQL - Add Dummy Data

In this section, we'll add some dummy data to your RDS MySQL database and connect to it from your EC2 instance.

### Connecting to the Database from EC2

To connect to your RDS MySQL database from your EC2 instance, you can use the `mysql` command-line client. Replace `endpointurl` with the actual endpoint URL of your RDS instance.

```bash
mysql -h endpointurl -P 3306 -u admin -p

# You'll be prompted to enter the password for the admin user (e.g., admin1234!).

Creating a New Database
You can create a new database called LIBRARY with the following commands:

  CREATE DATABASE LIBRARY;

#After creating the database, you can switch to it using:
USE LIBRARY;

#Creating a Table - Let's create a table named books with the following structure:

CREATE TABLE IF NOT EXISTS books (
	book_id INT,
	title VARCHAR(255) NOT NULL,
	publish_date DATE,
	description TEXT,
	PRIMARY KEY (book_id)
) ENGINE=INNODB;

# This table will store information about books, including an ID, title, publish date, and description.

## Viewing Tables and Data

You can list all the tables in the current database with:
SHOW TABLES;

To check the structure of the books table, you can use:
DESCRIBE books;

Finally, let's insert some dummy data into the books table:
INSERT INTO books (book_id, title, publish_date, description)
VALUES
    (1, 'Learn MySQL', CURDATE(), 'This is a book on MySQL.'),
    (2, 'Learn Oracle', CURDATE(), 'This is a book on Oracle.'),
    (3, 'Learn SQL Server', CURDATE(), 'This is a book on SQL Server.'),
    (4, 'Learn DB2', CURDATE(), 'This is a book on DB2.'),
    (5, 'Learn Aurora', CURDATE(), 'This is a book on Amazon Aurora.');

-- View the data in the books table
SELECT * FROM books;
```


# Create Replication Instance

we will create a replication instance in AWS Database Migration Service (DMS) to facilitate the data migration process.

### Create a Replication Instance

1. In the DMS dashboard, select **Replication instances** -> Click the **Create replication instance** button.

3. Configure the following settings for the replication instance:

   - **Name**: Enter a name for the replication instance (e.g., `dms2023rep`).
   - **Instance class**: Choose `dms.t2.micro` (or select "include previous generation instance classes" if necessary).
   - **Engine version**: Set it to `3.4.3`.
   - **Allocated storage**: Specify the storage size, e.g., `8GB`.
   - **VPC**: Select your default VPC.

4. Click "Create" to initiate the creation of the replication instance.

### Step 3: Advanced Settings (Optional)

If you have specific requirements, you can configure advanced settings such as:

   - **Availability Zone**: Choose the desired availability zone **us-east-1C**.
   - **VPC security group**: Use the default VPC security group or configure custom security groups as needed.

5. Once you've configured advanced settings (if necessary), click "Create" to create the replication instance.

Now, you have successfully created a replication instance in AWS DMS, which will be used for data migration tasks. You can proceed to create source and target endpoints and set up your migration tasks.

## DMS - Create Source Endpoint
we will create a source endpoint in AWS Database Migration Service (DMS) to connect to your RDS MySQL database for data migration.

### Step 2: Create a Source Endpoint
1. In the DMS dashboard, select "Endpoints" from the left navigation pane.-> Click the **Create endpoint** button.

3. Configure the following settings for the source endpoint:
   - **Endpoint type**: Select "source endpoint."
   - **Choose RDS instance**: Check the box next to your RDS instance (`DMS-mySQL-S3`).
   - **Source engine**: Choose "MYSQL."
   - **Access endpoint DB**: Select "Provide access information manually."

4. Provide the following connection details for your RDS MySQL database:

   - **Server name**: Enter the RDS endpoint URL.
   - **Port**: Set it to `3306`.
   - **SSL mode**: Choose "None."
   - **Username**: Enter the database username (e.g., `admin`).
   - **Password**: Enter the database password (e.g., `admin1234`).

5. Optionally, you can click "Test endpoint connection" to verify the connectivity between DMS and your RDS database. Ensure that the status shows "Successful."

6. Configure the VPC setting to use the **default VPC**

7. Select the replication instance you created earlier (e.g., **dms2023repli**) as the replication instance for this source endpoint.

8. Click **Create endpoint** to create the source endpoint.

Now, you have successfully created a source endpoint in AWS DMS, allowing DMS to connect to your RDS MySQL database. You can proceed to create a target endpoint and set up your database migration task.


## DMS - Create Target Endpoint

we will create a target endpoint in AWS Database Migration Service (DMS) to connect to an AWS S3 bucket, where your migrated data will be stored.

### Create a Target Endpoint

1. In the DMS dashboard, select **Endpoints** -> Click the **Create endpoint** button.
2. Configure the following settings for the target endpoint:
   - **Endpoint config**: Enter a unique identifier for the endpoint  **s3target**.
   - **Endpoint instance**: Choose "DMS-mySQL-S3" from the dropdown menu.
   - **Test endpoint**: Select "AWSS3" as the target engine.
3. In the "ARN" field, enter the **arn:aws:iam::your-account-id:role/mysqls3**. of the IAM role that grants DMS access to your S3 bucket. Replace **your-account-id** with your actual AWS account ID.
5. Provide the name of the S3 bucket where the migrated data will be stored (e.g., **mydmstos3**).
6. Optionally, click **Test endpoint connection** to verify the connectivity between DMS and your S3 bucket. Ensure that the status shows **Successful.**
7. Configure the VPC setting to use the **default VPC**.
8. Select the replication instance you created earlier (e.g., `dms2023repli`) as the replication instance for this target endpoint.
9. Click **Create endpoint** to create the target endpoint.

Now, you have successfully created a target endpoint in AWS DMS, allowing DMS to connect to your AWS S3 bucket, where data migrated from your source database will be stored. You can proceed to create a database migration task.


## DMS - Create Database Migration Task

In this section, we will create a database migration task in AWS Database Migration Service (DMS) to migrate data from your source RDS MySQL database to an AWS S3 bucket.

### Create a Database Migration Task

1. In the DMS dashboard, select **Database migration tasks** -> Click the **Create migration task** button.

2. Configure the following settings for the migration task:
   - **Task identifier**: Enter a unique identifier for the task (e.g., `MySQLS3`).
   - **Replication instance**: Select the replication instance you created earlier (e.g., `dms2023rep`).
   - **Source endpoint**: Choose "DMS-mySQL-S3" as the source endpoint.
   - **Target endpoint**: Choose the target endpoint you created for your S3 bucket (e.g., `s3target`).
   - **Migration type**: Select "Migrate existing data."

3. In the "Task settings" section, choose the following options:
   - **Editing mode**: Select "wizard."
   - **Target table preparation mode**: Choose "Drop tables on target."
   - **Include LOB columns in replication**: Select "Do not include LOB columns."

4. In the **Table mappings** section, configure the table mappings for your migration. Here's how to do it:
   - In **Editing mode** select "wizard."
   - Click **Add new selection rule**
   - For the schema name, enter `%library` (assuming your schema is named `library`).
   - For the table name, enter `%books` (assuming your table is named `books`).
   - Set the action to "Include."

5. After configuring the table mapping, click "Create task" to create the database migration task.

Now, you have successfully created a database migration task in AWS DMS. This task will migrate data from your RDS MySQL source database (schema `library`, table `books`) to your AWS S3 bucket specified in the target endpoint. The data will be loaded into the S3 bucket in CSV format, as configured.
You can monitor the progress of the migration task in the AWS DMS dashboard.

## S3 - Check the CSV File
we can verify that the data migration task has successfully generated the CSV file in your S3 bucket.
 
1. In the S3 dashboard, select the S3 bucket where you configured your AWS DMS target endpoint

2.  Inside the bucket, navigate to the location where the data should have been migrated based on your DMS task configuration.

4. Look for the CSV file containing your migrated data. The file name and location will depend on your DMS task configuration. You can open, download, or further process this CSV file as needed.

## Delete Resources
Once your migration is complete, it's important to clean up the AWS resources to avoid incurring unnecessary charges. Here is a list of resources you can consider deleting:

1. **EC2 Instances & Security Group**:
   - EC2 instances that you created for the migration.
   - The security group associated with those instances (e.g., `SG-mysql`).
2. **DMS Migration Task**: Delete the DMS database migration task (e.g., `MySQLS3`).
3. **DMS Endpoints**: Delete the source and target DMS endpoints you created.
4. **DMS Replication Instance**: Terminate or delete the DMS replication instance (e.g., `dms2023rep`).
5. **RDS Database**: RDS MySQL database (`db-mysql-s3`)
6. **RDS Subnet Groups**: Delete any RDS subnet groups
7. **S3 Buckets**: Remove any S3 buckets created for migration tasks
8. **IAM Role**: IAM role for DMS access to S3, you can delete it 


By cleaning up these resources, you can manage your AWS environment efficiently and reduce costs associated with unused resources.

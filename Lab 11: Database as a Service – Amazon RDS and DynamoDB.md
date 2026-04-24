# Implementing Cloud Database Services Using Amazon RDS and DynamoDB

---

**Objectives**

 - Deploy a managed relational database using Amazon RDS
 - Create and interact with a NoSQL database using DynamoDB
 - Understand the Database as a Service model in cloud environments
 - Configure database security, automated backups, and monitoring
 - Compare relational (SQL) and NoSQL database services on AWS

---

**Theory**

Cloud databases remove the burden of managing physical hardware, applying patches, scheduling backups, and handling scaling manually. AWS provides fully managed database services designed for different workload types.

**Amazon RDS (Relational Database Service)** supports managed SQL engines including MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Aurora. It handles automated backups, OS and engine patching, and Multi-AZ failover for high availability. Read replicas can be added to distribute read traffic and improve performance.

**Amazon DynamoDB** is a fully managed NoSQL database that stores data in key value and document formats. It delivers single digit millisecond response times at any scale and operates serverlessly, scaling capacity up or down automatically. Billing is either per request or based on provisioned throughput. It is widely used for high-throughput workloads such as gaming leaderboards, IoT data ingestion, and mobile application backends.

Both services are foundational components in cloud application architecture. They provide persistent data storage for web services, scientific computing, and business systems. In multi tenant environments, data isolation between tenants is a key security concern, enforced through encryption, network controls, and access policies.

---

**Procedure**

**Part A: Amazon RDS (Relational Database)**

**Step 1: Create an RDS Database Instance**

    1. Open the AWS Management Console and navigate to Amazon RDS.
    2. Click "Create database."
    3. Set the creation method to "Standard Create."
    4. Select MySQL as the engine and choose the latest available version.
    5. Under templates, select "Free Tier."
    6. Set the DB instance identifier to `cloud-lab-db`.
    7. Set the master username to `admin` and create a strong password.
    8. Choose `db.t3.micro` as the instance class.
    9. Set storage to 20 GB using the gp2 storage type.
    10. Enable public access (for this lab only; this setting should never be used in production).
    11. Under the VPC security group, create a new group and add an inbound rule to allow TCP traffic on port 3306 from your IP address.
    12. Click "Create database."

>*Take a screenshot of the database creation configuration page.*



**Step 2: Wait for the Database to Become Available**

Monitor the RDS dashboard until the database status changes to "Available." This typically takes 5 to 10 minutes. Once available, note the endpoint address shown in the instance details.

Take a screenshot of the RDS dashboard showing the "Available" status and the endpoint.



**Step 3: Connect to the RDS Instance from Your EC2 Instance**

SSH into your EC2 instance, then run the following commands:

```bash name=connect-rds.sh
sudo yum install mysql -y
mysql -h <rds-endpoint> -u admin -p
```

Replace `<rds-endpoint>` with the endpoint copied from the RDS dashboard. Enter your password when prompted.



**Step 4: Create a Database, Table, and Insert Records**

Once connected to MySQL, run the following SQL commands:

```sql name=setup-database.sql
CREATE DATABASE cloud_lab;
USE cloud_lab;

CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    course VARCHAR(100),
    enrollment_date DATE DEFAULT CURRENT_DATE
);

INSERT INTO students (name, email, course) VALUES
('Alice Johnson', 'alice@example.com', 'Cloud Computing'),
('Bob Smith', 'bob@example.com', 'Cloud Security'),
('Charlie Brown', 'charlie@example.com', 'Virtualization');

SELECT * FROM students;
```

>*Take a screenshot of the MySQL terminal showing the table creation and the query results.*



**Step 5: Configure Automated Backups**

    1. In the RDS console, select your database instance.
    2. Click "Modify."
    3. Under the "Backup" section, set the backup retention period to 7 days.
    4. Select a preferred backup window.
    5. Apply the changes.

>*Take a screenshot of the backup configuration settings.*



**Step 6: Create a Read Replica**

    1. In the RDS console, select your database instance.
    2. Click "Actions," then select "Create read replica."
    3. Review the available configuration options.

>Note: Read replicas may not be available under the Free Tier. If the option is unavailable, document the steps and the restriction.

>*Take a screenshot of the read replica creation page.*

---

**Part B: Amazon DynamoDB (NoSQL Database)**

**Step 7: Create a DynamoDB Table**

    1. Navigate to Amazon DynamoDB in the AWS console.
    2. Click "Create table."
    3. Set the table name to `CloudLabCourses`.
    4. Set the partition key to `CourseID` with type String.
    5. Set the sort key to `UnitNumber` with type Number.
    6. Leave table settings at their defaults, which will use on-demand capacity mode.
    7. Click "Create table."

*Take a screenshot of the table creation configuration page.*

---

**Step 8: Add Items to the Table**

    1. In the DynamoDB console, open the `CloudLabCourses` table.
    2. Click "Explore table items," then click "Create item."
    3. Switch to the JSON view and enter the following:

```json name=dynamodb-item-1.json
{
  "CourseID": "CC101",
  "UnitNumber": 1,
  "UnitName": "Introduction to Cloud Computing",
  "Hours": 6,
  "Topics": ["Evolution", "Characteristics", "Types", "Benefits"]
}
```

    4. Save the item.
    5. Repeat the process to add 3 to 4 more items representing different course units.

*Take a screenshot of the table showing the inserted items.*

---

**Step 9: Query and Scan the Table**

    1. In the "Explore table items" view, run a query using `CourseID = CC101` as the partition key.
    2. Run a scan with a filter expression to return only items where `Hours > 8`.
    3. Review the returned results.

*Take a screenshot of both the query and scan results.*

---

**Comparing RDS and DynamoDB**

| Feature | RDS (MySQL) | DynamoDB |
|---|---|---|
| Data model | Relational (tables and rows) | Key value and document |
| Schema | Fixed schema | Flexible schema |
| Query language | SQL | API based (Query and Scan) |
| Scaling | Vertical (instance resize) | Horizontal (automatic) |
| Use case | Complex queries and transactions | High throughput, low latency workloads |
| Managed backups | Yes | Yes (continuous) |
| Pricing model | Instance based | Per request or provisioned |


---

**Results**

 - An RDS MySQL instance was deployed, connected to from an EC2 instance, and populated with student records.
 - SQL queries were executed successfully against a cloud-managed relational database.
 - A DynamoDB NoSQL table was created and populated with items using a flexible schema.
 - Both databases were configured with appropriate security settings and backup policies.
 - A clear understanding was established of when each database type is the appropriate architectural choice.

---

**Discussion and Conclusion**

This lab demonstrated the Database as a Service model on AWS. Amazon RDS removed the need to manually install, patch, or back up MySQL by automating those operational tasks entirely. For applications that require complex joins, multi table transactions, and a well defined relational structure, such as enterprise business applications, RDS is the appropriate choice.

DynamoDB offers a serverless, horizontally scalable alternative for applications that need low latency access to semi structured or variable data, including IoT platforms, gaming backends, and mobile applications. Both services address multi-tenant data security through encryption at rest using AWS KMS, network isolation via VPC, and fine-grained access control through IAM policies.

The decision between SQL and NoSQL is an architectural one, driven by data access patterns, consistency requirements, and expected scale. Understanding this distinction is essential when designing cloud native applications.

---

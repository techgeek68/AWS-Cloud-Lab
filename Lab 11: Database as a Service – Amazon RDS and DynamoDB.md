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

Cloud databases remove the burden of managing physical hardware, applying patches, scheduling backups, and manually scaling. AWS provides fully managed database services designed for different workload types.

**Amazon RDS (Relational Database Service)** 

It supports managed SQL engines, including MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Aurora. It handles automated backups, OS and engine patching, and Multi-AZ failover for high availability. Read replicas can be added to distribute read traffic and improve performance.

**Amazon DynamoDB** 

It is a fully managed NoSQL database that stores data in key value and document formats. It delivers single digit millisecond response times at any scale and operates serverlessly, scaling capacity up or down automatically. Billing is either per request or based on provisioned throughput. It is widely used for high-throughput workloads such as gaming leaderboards, IoT data ingestion, and mobile application backends.

Both services are foundational components in cloud application architecture. They provide persistent data storage for web services, scientific computing, and business systems. In multi tenant environments, data isolation between tenants is a key security concern, enforced through encryption, network controls, and access policies.

---

**Procedure**

**Part A: Amazon RDS (Relational Database)**

**Step 1: Create an RDS Database Instance**

 1. Open the AWS Management Console and navigate to **Aurora and RDS** using the search bar.
 2. In the left sidebar, click **Databases**, then click **Create database**.
 3. Under **Engine type**, select **MySQL**.
 4. Under **Choose a database creation method**, select **Full configuration**.
 5. Under **Templates**, select **Dev/Test**.
 6. Under **Availability and durability**, select **Single-AZ DB instance deployment (1 instance)**.
 7. Under **Settings**, configure the following:
    - DB instance identifier: `cloud-lab-db`
    - Master username: `admin`
    - Credentials management: select **Self managed**
    - Master password: create a strong password (e.g., `Admin12345!`) and note it down
    - Confirm master password: re-enter the same password
 8. Under **Instance configuration**:
    - Check **Burstable classes (includes t classes)**
    - Instance type: select **db.t3.micro**
 9. Under **Storage**:
    - Storage type: **gp2**
    - Allocated storage: `20` GiB
    - Expand **Additional storage configuration** > uncheck **Enable storage autoscaling**
 10. Under **Connectivity**:
     - Compute resource: **Don't connect to an EC2 compute resource**
     - VPC: **Default VPC**
     - Public access: **Yes**
     - VPC security group: select **Create new**
     - New VPC security group name: `DB-security-group`
     - Availability Zone: **No preference**
    
 11. Scroll down to **Monitoring**:
     - Uncheck **Enable Performance Insights**
     - Uncheck **Enable Enhanced Monitoring**
    
 12. Scroll down and expand **Additional configuration**:
     - Backup retention period: **7 days**
     - Choose a preferred backup window (Start time and Duration)
     - Uncheck **Enable delete protection**
     - Leave all other settings as the default
     
 13. Click **Create database**. You will be automatically redirected to the **Databases** dashboard. Monitor the status of `cloud-lab-db`; it will initially show **Creating**. Wait **5 to 10 minutes** until the status changes to **Available** (shown in green). Once available, click on `cloud-lab-db` to open the instance details.

 14. Click the **Connectivity & security** tab and copy the **Endpoint** address (it looks like: `cloud-lab-db.xxxxxxxxx.us-east-1.rds.amazonaws.com`). Save this; you will need it in Step 3.
   
>*Screenshot: Take a screenshot of the RDS dashboard showing the **Available** status and the endpoint address [Mandatory].*

<img width="1458" height="552" alt="Screenshot 2026-04-27 at 8 46 28 AM" src="https://github.com/user-attachments/assets/2639112a-9613-49c9-9cca-84bfd5a48ccb" />

<img width="1452" height="713" alt="Screenshot 2026-04-27 at 8 54 46 AM" src="https://github.com/user-attachments/assets/cd3d52d8-483f-445a-84bd-e1af1a53964e" />


**Step 2: Launch EC2 Instance and Edit Inbound Rule to the Security Groups**

1. Navigate into **EC2**
 
 - Network & Security > Security Groups
   - Basic details
   - Security group name: `DB-lab-sg`
   - Descriptions: Allows SSH from the developer.
   - VPC: leave default
   - Inbound rule:
    - Type: SSH, Protocol: TCP, Port range: 22, Source type: `Anywhere IPv4``0.0.0.0/0`
   - Outbound rules
    - Leave default
   - Create security group 
 - EC2 > Instances > Click on the **Launch instance**
   - Name: MyDatabaseServer
   - Application and OS Images: Red Hat
   - Instance type: t2.micro
   - Key pair: Create one or use a previously created one.
   - Network settings
   - Select existing security group
    - Common security groups: `DB-lab-sg`
   - Configure storage: Leave default
   - Click on the **Launch instance**
      
**The following step is required before you can connect to the database.**
  1. In the AWS Console, navigate to EC2 > Security Groups
  2. Find and click on **DB-security-group**.
  3. Click the **Inbound rules** tab > click **Edit inbound rules**.
  4. Click **Add rule** and set:
    - Type: `MySQL/Aurora`
    - Protocol: `TCP`
    - Port range: `3306`
    - Source: select Custom > type the name `DB-lab-sg` and select it
  5. Click **Save rules**.

**Step 3: Connect to the RDS Instance from Your EC2 Instance**
 
1. Open your terminal and SSH into your EC2 instance:
```bash
# First, set correct permissions on your key file
chmod 400 CloudLab.pem

# Then SSH in
ssh -i CloudLab.pem ec2-user@<your-ec2-public-ip>
```
>Sample:

<img width="1453" height="615" alt="Screenshot 2026-04-27 at 11 16 26 AM" src="https://github.com/user-attachments/assets/1d264496-198a-4bed-af48-558407b486e9" />

2. Register your Red Hat Server
```bash
sudo rhc connect
```
>Enter your Red Hat Academy Login username
>Enter your Red Hat Academy Login Password

>Sample

<img width="1456" height="268" alt="Screenshot 2026-04-27 at 11 21 45 AM" src="https://github.com/user-attachments/assets/6dc8b6fd-c5f4-446a-96ab-00dd7a68fd9a" />

3. Install the MySQL client:
```bash
sudo yum install mysql* -y
```
3. Connect to your RDS instance (replace `<rds-endpoint>` with the endpoint you copied in Step 2):
```bash
mysql -h <rds-endpoint> -u admin -p
```
4. Enter your password when prompted. You should see the `mysql>` prompt if the connection is successful.


**Step 4: Create a Database, Table, and Insert Records**

 - Once connected at the `mysql>` prompt, run the following SQL commands one by one:
```sql
DROP DATABASE your_database_name;
USE cloud_lab;
```
 - Create the table
```sql
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    course VARCHAR(100),
    enrollment_date DATE DEFAULT (CURRENT_DATE)
);
```
 - Insert the records:
```sql

INSERT INTO students (name, email, course) VALUES
('Natasha', 'natasha@example.com', 'Cloud Computing'),
('David', 'david@example.com', 'Cloud Security'),
('Kamal', 'kamal@example.com', 'Virtualization');

SELECT * FROM students;
```
 - Verify the records:
```sql
SELECT * FROM students;
```

>Delete:` DELETE FROM students WHERE id IN (4, 5, 6);`

>Edit: `UPDATE students SET email = 'kamal@gmail.com' WHERE id = 9;` `UPDATE students SET course = 'Cloud Computing' WHERE id = 8;`

>*Screenshot: Take a screenshot of the MySQL terminal showing the table creation and the `SELECT *` query results [Mandatory].*

<img width="1455" height="154" alt="Screenshot 2026-04-27 at 1 00 49 PM" src="https://github.com/user-attachments/assets/1f899529-1960-4eaf-bcef-96d3d388c937" />



**Step 5: Configure Automated Backups [Optional]**
 1. In the RDS console, click on your database instance `cloud-lab-db`.
 2. Click the **Modify** button (top right).
 3. Scroll down to the **Backup** section and configure:
   - Backup retention period: **7 days**
   - Backup window: select any preferred time window (Start time 00:30 UTC, Duration 1 hours)
 4. Scroll down and click **Continue**.
 5. Under **Scheduling of modifications**, select **Apply immediately**.
 6. Click **Modify DB instance**.


**Step 6: Create a Read Replica [Optional]**
 1. In the RDS console, click on your database instance `cloud-lab-db`.
 2. Click **Actions** (top right dropdown) > select **Create read replica**.
 3. Review the available configuration options such as destination region, instance class, and storage.

---

**Part B: Amazon DynamoDB (NoSQL Database)**


**Step 1: Create a DynamoDB Table**

1. In the AWS Management Console, search for and navigate to **DynamoDB**.
2. In the left sidebar, click **Tables**, then click **Create table**.
3. Configure the table:
   - Table name: `CloudLabCourses`
   - Partition key: `CourseID` > type: **String**
   - Sort key: `UnitNumber` > type: **Number**
4. Under **Table settings**, leave everything as default (this uses on demand capacity mode).
5. Click **Create table**.
   

**Step 2: Add Items to the Table**

1. In the DynamoDB console, click on **Tables** > click on `CloudLabCourses`.
2. Click **Explore table items** > click **Create item**.
3. Click the **JSON view** toggle (top right of the editor).
4. Clear the existing content and paste the following, then click **Create item**:

```json
{
  "CourseID": {"S": "CC101"},
  "UnitNumber": {"N": "1"},
  "UnitName": {"S": "Introduction to Cloud Computing"},
  "Hours": {"N": "6"},
  "Topics": {"L": [
    {"S": "Evolution"},
    {"S": "Characteristics"},
    {"S": "Types"},
    {"S": "Benefits"}
  ]}
}
```

5. Repeat the process to add 3 more items. Click **Create item** each time and use the following:

**Item 2:**
```json
{
  "CourseID": {"S": "CC101"},
  "UnitNumber": {"N": "2"},
  "UnitName": {"S": "Cloud Architecture"},
  "Hours": {"N": "9"},
  "Topics": {"L": [
    {"S": "IaaS"},
    {"S": "PaaS"},
    {"S": "SaaS"},
    {"S": "Deployment Models"}
  ]}
}
```

**Item 3:**
```json
{
  "CourseID": {"S": "CC102"},
  "UnitNumber": {"N": "1"},
  "UnitName": {"S": "Cloud Security Fundamentals"},
  "Hours": {"N": "10"},
  "Topics": {"L": [
    {"S": "IAM"},
    {"S": "Encryption"},
    {"S": "Compliance"},
    {"S": "Shared Responsibility"}
  ]}
}
```

**Item 4:**
```json
{
  "CourseID": {"S": "CC102"},
  "UnitNumber": {"N": "2"},
  "UnitName": {"S": "Cloud Storage Solutions"},
  "Hours": {"N": "8"},
  "Topics": {"L": [
    {"S": "S3"},
    {"S": "EBS"},
    {"S": "EFS"},
    {"S": "Glacier"}
  ]}
}
```

>*Screenshot: Take a screenshot of the Explore table items view showing all inserted items. [Mandatory].*

<img width="1470" height="529" alt="Screenshot 2026-04-27 at 12 43 09 PM" src="https://github.com/user-attachments/assets/4a1c2fbe-d452-4263-abf3-b38dfad2beb4" />


**Step 9: Query and Scan the Table**

1. In the `CloudLabCourses` table, click **Explore items**.

**Run a Query:**

2. Under **Scan or query items**, select **Query**.
3. Set the partition key field:
   - Attribute `CourseID`
   - Value `CC101`
4. Click **Run** and review the results - it should return all units under CC101.


**Run a Scan with Filter:**

5. Switch to **Scan** mode.
6. Expand **Filters** and add a filter:
   - Attribute name: `Hours`
   - Condition: **Greater than**
   - Type: **Number**
   - Value: `8`
7. Click **Run** - only items where Hours > 8 should appear (Units with 9 and 10 hours).

>*Screenshot: Take a screenshot of both the query results and the scan filter results [Mandatory]*.

<img width="1458" height="716" alt="Screenshot 2026-04-27 at 12 51 37 PM" src="https://github.com/user-attachments/assets/1778cc93-3f09-4c49-b373-5a4c2d92e7cb" />

<img width="1470" height="608" alt="Screenshot 2026-04-27 at 12 49 36 PM" src="https://github.com/user-attachments/assets/5ac1ae6c-b406-4ee5-bb28-0644870c4501" />


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
 - SQL queries were executed successfully against a cloud managed relational database.
 - A DynamoDB NoSQL table was created and populated with items using a flexible schema.
 - Both databases were configured with appropriate security settings and backup policies.
 - A clear understanding was established of when each database type is the appropriate architectural choice.

---

**Discussion and Conclusion**

This lab demonstrated the Database as a Service model on AWS. Amazon RDS removed the need to manually install, patch, or back up MySQL by automating those operational tasks entirely. For applications that require complex joins, multi table transactions, and a well defined relational structure, such as enterprise business applications, RDS is the appropriate choice.

DynamoDB offers a serverless, horizontally scalable alternative for applications that need low latency access to semi structured or variable data, including IoT platforms, gaming backends, and mobile applications. Both services address multi-tenant data security through encryption at rest using AWS KMS, network isolation via VPC, and fine-grained access control through IAM policies.

The decision between SQL and NoSQL is an architectural one, driven by data access patterns, consistency requirements, and expected scale. Understanding this distinction is essential when designing cloud native applications.

---

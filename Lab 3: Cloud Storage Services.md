## Implementing Cloud Storage using Amazon S3 and EBS

**Objectives:**
  - Create and configure Amazon S3 buckets with different storage classes
  - Upload, manage, and access objects in S3
  - Configure S3 bucket policies and static website hosting
  - Understand block storage using EBS volumes
  - Compare object storage (S3) vs. block storage (EBS)

**Theory:**

Cloud storage removes the need for on-premises storage infrastructure. AWS provides multiple storage services suited to different use cases.

**Amazon S3 (Simple Storage Service):**
  * Object storage with 99.999999999% durability. Data is stored as objects inside buckets. Supported storage classes include S3 Standard, S3 Intelligent Tiering, and S3 Glacier for archival. Common use cases include backups, media hosting, static websites, and data lakes.

**Amazon EBS (Elastic Block Store):**
  * Block level storage volumes attached to EC2 instances. Functions like a virtual hard disk. Supports SSD types (gp3, io2) and HDD types (st1, sc1).

**Amazon EFS (Elastic File System):**
  * A managed NFS file system that can be shared across multiple EC2 instances.

S3 uses a flat namespace with buckets acting as containers and objects as the stored files. A unique key identifies each object. S3 also supports versioning, lifecycle policies, encryption, and access control.

---

**Procedure:**

**Part A: Amazon S3**

**Step 1:** Navigate to S3 and click "Create Bucket". Configure the following:
  - AWS Region: US East (N. Virginia) us-east-1
  - Bucket name: `cloud-lab-storage-<your-class-roll-number>` (must be globally unique)
  - Object Ownership: ACLs disabled
  - Leave "Block all public access" checked for now
  - Bucket Versioning: Disable

**Step 2:** Click "Create Bucket" and open the newly created bucket.

>*Screenshot: S3 bucket created. [Mandatory]*

>Sample:

<img width="1459" height="598" alt="Screenshot 2026-04-13 at 6 13 43 AM" src="https://github.com/user-attachments/assets/f843fd74-e7f8-4c48-b7b2-6cb46cc657eb" />


**Step 3:** Upload files to the bucket:
  - Click "Upload" and add files (for example, index.html, an image, or a PDF)
  - Note that the storage class defaults to "S3 Standard"

>*Screenshot: S3 bucket with uploaded objects showing storage class. [Mandatory]*

> Sample:

<img width="1459" height="371" alt="Screenshot 2026-04-13 at 6 17 59 AM" src="https://github.com/user-attachments/assets/ccc3854f-455d-4231-9d5f-ded240aaa28a" />


**Step 4:** Enable Versioning:
  - Bucket > Properties > Bucket Versioning > Enable
  - Upload the same file again with modified content (Compress the image using https://tinypng.com/index.html & upload again)
  - Toggle "Show versions" to view the stored versions

>*Screenshot: S3 versioning showing multiple versions of the same object. [Mandatory]*

>Sample:

<img width="1459" height="371" alt="Screenshot 2026-04-13 at 6 17 59 AM" src="https://github.com/user-attachments/assets/ab8c66f0-4a8e-4df4-825f-ba72992830c7" />


<img width="1459" height="362" alt="Screenshot 2026-04-13 at 6 36 54 AM" src="https://github.com/user-attachments/assets/99f9745e-4b1f-4510-9b52-2e99ffee36bc" />


**Step 5:** Configure Static Website Hosting:
  - Bucket > Properties > Static website hosting > Enable
  - Index document: `index.html`
  - Error document: `error.html`

>*Screenshot: Static website hosting configuration.[Mandatory]*

>Sample:

<img width="1457" height="668" alt="Screenshot 2026-04-13 at 6 42 35 AM" src="https://github.com/user-attachments/assets/b8b885f1-ba82-41fa-9e92-d6b4131db24e" />


**Step 6:** Update the bucket policy to allow public read access for the website:
  - Bucket > Permissions > Uncheck "Block all public access" > Confirm
  - Add the following bucket policy:

```json name=bucket-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloud-lab-storage-<your-unique-id>/*"
    }
  ]
}
```

>*Screenshot: Bucket policy editor with the public read policy applied. [Mandatory]**

>Sample:

<img width="1455" height="602" alt="Screenshot 2026-04-13 at 6 58 16 AM" src="https://github.com/user-attachments/assets/bcf4a339-c669-4c05-ae0f-85cb50644621" />



**Step 7:** Create an `index.html` file with the following content:

```html name=index.html
<!DOCTYPE html>
<html>
<head><title>Cloud Storage Lab</title></head>
<body>
  <h1>Welcome to S3 Static Website Hosting!</h1>
  <p>This page is served from Amazon S3.</p>
  <p>Full Name: Samriddhi College</p>
  <p>Roll Number: 502</p>
</body>
</html>
```

Upload the file and open the S3 website endpoint URL in a browser.

> *Screenshot: Browser showing the S3 hosted static website. [Mandatory]*

> Sample:

<img width="1467" height="313" alt="Screenshot 2026-04-13 at 7 02 17 AM" src="https://github.com/user-attachments/assets/9d3a15ca-432a-4753-96b8-47290aacfd58" />


**Step 8:** Configure a Lifecycle Rule:
  - Bucket > Management > Create lifecycle rule
  - Rule Name: `TransitionToGlacierExpireInYear`
  - Prefix: Limit the rule to objects in a folder or specific path (2026/uploads/)
  - Tags: Glacier
  - Rule: Transition current versions of objects between storage classes. (Transition objects to S3 Glacier after 90 days, expire after 365 days)
    - Glacier Flexible Retrieval (formerly Glacier), Days after object creation: 365

> *Screenshot: Lifecycle rule configuration. [Mandatory]*

> Sample:

<img width="1458" height="394" alt="Screenshot 2026-04-13 at 7 30 22 AM" src="https://github.com/user-attachments/assets/cadd1ee8-8a4d-435b-b3dc-d487a3db1b68" />

---

**Part B: Amazon EBS**

**Prerequities** : Navigate to EC2 > Launch Instance
  - Name: EBSLab
  - Application and OS Images: Amazon Linux
  - Architecture: x86 (64-bit)
  - Instance type: t2.micro
  - Key pair: CloudLab
  - Security Groups: Select existing security group
  - Configure storage: 8 GiB, gp3
- Launch Instance

>Sample:

<img width="1459" height="266" alt="Screenshot 2026-04-13 at 8 12 25 AM" src="https://github.com/user-attachments/assets/5b165755-2731-4dd0-a6cb-24ad062b2eb1" />

  
**Step 9:** EC2 > Volumes > Create Volume with the following settings:
  - Volume type: General purpose SSD (gp3)
  - Size: 10 GB
  - Availability Zone: Same as your running EC2 instance

>*Screenshot: EBS volume creation page. [Mandatory]*

>Sample:

![Screenshot 2026-04-13 at 8 14 42 AM](https://github.com/user-attachments/assets/7a1ff713-96c7-42df-a931-6d41f2683448)


**Step 10:** Attach the volume to your EC2 instance:
  - Select the volume > Actions > Attach Volume > Select your instance > Device name: /dev/sdb
  - SSH into the instance and mount the volume using the following commands:

<img width="960" height="311" alt="Screenshot 2026-04-13 at 8 17 45 AM" src="https://github.com/user-attachments/assets/5f772b3f-4f36-4239-a0b0-e156ee367f3e" />


```bash name=mount-ebs.sh
sudo lsblk
```
```bash name=mount-ebs.sh
sudo mkfs -t xfs /dev/xvdb
```
```bash name=mount-ebs.sh
sudo mkdir /data
```
```bash name=mount-ebs.sh
sudo mount /dev/xvdb /data
```
```bash name=mount-ebs.sh
echo "Hello from EBS" | sudo tee /data/test.txt
cat /data/test.txt
```

>*Screenshot: Terminal showing the mounted EBS volume and file operations. [Mandatory]*

>Sample:

<img width="705" height="105" alt="Screenshot 2026-04-13 at 8 22 02 AM" src="https://github.com/user-attachments/assets/5bdf612a-3766-4839-97b1-915fc00ceb87" />


<img width="651" height="175" alt="Screenshot 2026-04-13 at 8 22 16 AM" src="https://github.com/user-attachments/assets/ee505f79-6746-42ff-a4b0-b3d94bf3c727" />


<img width="684" height="86" alt="Screenshot 2026-04-13 at 8 22 35 AM" src="https://github.com/user-attachments/assets/3340b3f7-e376-4d17-a8f3-8eca1e987de1" />


<img width="708" height="122" alt="Screenshot 2026-04-13 at 8 22 46 AM" src="https://github.com/user-attachments/assets/86dc86d8-01ea-4845-b38a-a80ef0a5df87" />

---

**Results:**
  - S3 bucket created with versioning, lifecycle policies, and static website hosting configured
  - Static website successfully hosted on S3
  - EBS volume created, attached to an EC2 instance, formatted, and mounted
  - Both object storage (S3) and block storage (EBS) functionality were demonstrated

---

**Discussion and Conclusion:**

This lab covered two core AWS storage types. Amazon S3 provides highly durable, scalable object storage suited to unstructured data, backups, and static content delivery. Versioning protects against accidental deletion, and lifecycle rules reduce costs by moving data to cheaper storage tiers over time. EBS provides block level storage that acts as a persistent virtual disk for EC2 instances, making it a better fit for databases and file systems that require low latency access. Knowing when to use object storage versus block storage is a practical skill for designing cloud architectures that are both efficient and cost effective.

---

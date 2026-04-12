# Title: Understanding Cloud Service Models using AWS EC2, Elastic Beanstalk, and Amazon WorkDocs

**Objectives:**
  - Differentiate between IaaS, PaaS, and SaaS service models on AWS
  - Launch an EC2 instance (IaaS)
  - Deploy an application using Elastic Beanstalk (PaaS)
  - Access a SaaS application through AWS
  - Compare the level of control, flexibility, and management responsibility across models

**Theory:**

Cloud service models define the level of abstraction and management responsibility assigned to the provider versus the user.

  **Infrastructure as a Service (IaaS):** Provides raw computing resources such as virtual machines, storage, and networks. The user manages the OS, middleware, runtime, and applications. AWS Example: Amazon EC2.

  **Platform as a Service (PaaS):** Provides a managed platform for application development and deployment. The provider handles infrastructure, OS, and runtime. Users focus on code and data. AWS Example: AWS Elastic Beanstalk.

  **Software as a Service (SaaS):** Fully managed applications delivered over the internet. Users consume the software without managing any underlying infrastructure. AWS Examples: Amazon WorkDocs, Amazon Chime.

The Shared Responsibility Model shifts accordingly: in IaaS, the customer manages almost everything above the hypervisor; in PaaS, the platform handles most operations; in SaaS the provider manages nearly everything.


**Procedure:**

**Part A: IaaS: Launching an EC2 Instance**

**Step 1:** Log in to the AWS Console and navigate to the EC2 service.

**Step 2:** Click "Launch Instance" and configure the following:
  - Name: `IaaS-Lab-Instance`
  - AMI: Amazon Linux 2023
  - Instance type: t2.micro
  - Key pair: Create a new key pair and download the .pem file
  - Network settings: Allow SSH (port 22) and HTTP (port 80)
  - Storage: 8 GB gp3 (default)

>*Screenshot: EC2 instance configuration summary page. [Mandatory]*

**Step 3:** Click "Launch Instance" and wait for the instance state to show "Running".

*Screenshot: EC2 dashboard showing running instance with public IP [Mandatory]*

**Step 4:** Connect to the instance via SSH:

```bash name=connect.sh
chmod 400 your-key.pem
ssh -i "your-key.pem" ec2-user@<public-ip-address>
```

**Step 5:** Install a web server to demonstrate full control over the instance:

>Note: Before running the command below, make sure to replace the Full Name and Class Roll Number with your own details [Mandatory].

```bash name=setup-httpd.sh
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
echo "<html>
<head><title>EC2 Page</title></head>
<body>
<h1>Hello from IaaS - EC2 Instance</h1>
<p><strong>Full Name:</strong> Your Full Name</p>
<p><strong>Class Roll No:</strong> Your Roll Number</p>
</body>
</html>" | sudo tee /var/www/html/index.html
```

*Screenshot: Browser showing the web page served from the EC2 public IP [Mandatory]*


**Part B: PaaS: Deploying with Elastic Beanstalk**

**Step 6:** Navigate to Elastic Beanstalk and click "Create Application". Configure the following:
- Application name: `PaaS-Lab-App`
- Platform: Python (or Node.js)
- Application code: Select "Sample application"

*Screenshot: Elastic Beanstalk application creation page [Mandatory]*

**Step 7:** Click "Create Application" and wait for the environment to launch (5 to 10 minutes). Elastic Beanstalk automatically provisions the EC2 instance, load balancer, and Auto Scaling group.

*Screenshot: Elastic Beanstalk dashboard showing environment health as "OK" [Mandatory]*

**Step 8:** Click the environment URL to view the deployed sample application.

*Screenshot: Sample application running via the Elastic Beanstalk URL [Mandatory]*


**Part C: SaaS – Accessing Amazon WorkDocs**

**Step 9:** Navigate to Amazon WorkDocs and create a new site (or use Amazon Chime as an alternative SaaS example).
  - Register with your email and create a site name
  - Upload a document and share it with another user

*Screenshot: Amazon WorkDocs dashboard showing an uploaded and shared document [Mandatory]*

**Step 10:** Compare the three models across the following aspects:

| Aspect | IaaS (EC2) | PaaS (Beanstalk) | SaaS (WorkDocs) |
|---|---|---|---|
| User manages OS | Yes | No | No |
| User manages App | Yes | Yes | No |
| Scalability | Manual/Configured | Automatic | Automatic |
| Flexibility | Highest | Medium | Lowest |
| Setup complexity | High | Medium | Low |

**Results:**
  - Successfully launched and configured an EC2 instance (IaaS) with a custom web server
  - Deployed a sample application using Elastic Beanstalk (PaaS) with no infrastructure management required
  - Accessed Amazon WorkDocs (SaaS) as a fully managed service
  - Observed the different levels of management responsibility across all three service models

**Discussion and Conclusion:**

This lab demonstrated the three fundamental cloud service models in practice. EC2 (IaaS) provided full control over the virtual machine, requiring manual OS configuration and software installation. Elastic Beanstalk (PaaS) abstracted the infrastructure layer completely; only the application code was supplied, and the platform handled provisioning, load balancing, and scaling. WorkDocs (SaaS) required no technical setup at all. This progression reflects the core trade-off between control and convenience. Organizations select the appropriate model based on their needs: IaaS for maximum flexibility, PaaS for faster development cycles, and SaaS for ready-to-use solutions.

---

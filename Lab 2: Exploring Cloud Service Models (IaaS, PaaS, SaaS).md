# Understanding Cloud Service Models using AWS EC2, AWS Lambda, and Amazon WorkDocs

---

**Objectives:**
  - Differentiate between IaaS, PaaS, and SaaS service models on AWS
  - Launch an EC2 instance (IaaS)
  - Deploy and invoke a serverless function using AWS Lambda (PaaS)
  - Access a SaaS application through AWS
  - Compare the level of control, flexibility, and management responsibility across models

---

**Theory:**

Cloud service models define the level of abstraction and management responsibility assigned to the provider versus the user.

**Infrastructure as a Service (IaaS):** 

  Provides raw computing resources such as virtual machines, storage, and networks. The user manages the OS, middleware, runtime, and applications. In IaaS, the customer manages almost everything above the hypervisor. For example: Amazon EC2.

**Platform as a Service (PaaS):** 

  Provides a managed platform for application development and deployment. The provider handles infrastructure, OS, and runtime. Users focus on code and data. In PaaS, the platform handles most operations. For example: AWS Lambda.

**Software as a Service (SaaS):** 

  Fully managed applications delivered over the internet. Users consume the software without managing any underlying infrastructure. In SaaS, the provider manages nearly everything. For example: Amazon WorkDocs, Amazon Chime.

---

**Procedure:**

**Part A: IaaS Launching an EC2 Instance**

**Step 1:** Log in to the AWS Console and navigate to the EC2 service.

**Step 2:** Click "Launch Instance" and configure the following:
- Name: `IaaS-Lab-Instance`
- AMI: Amazon Linux 2023
- Instance type: t2.micro
- Key pair: Create a new key pair `CloudLab` and download the .pem file
- Network settings: Allow SSH (port 22) and HTTP (port 80)
  - VPC: `default`
  - Create a security group
    - Name: My Firewall
    - Description: Allowing SSH and HTTP Services
    - Inbound rule: SSH, TCP, 22, Anywhere and HTTP, TCP, 80, Anywhere
- Storage: 8 GB gp3 (default)


**Step 3:** Click "Launch Instance" and wait for the instance state to show "Running".

> *Screenshot: EC2 dashboard showing running instance with public IP [Mandatory]*

> Sample:

<img width="1467" height="716" alt="Screenshot 2026-04-12 at 10 30 32 AM" src="https://github.com/user-attachments/assets/385b85b1-f81f-4870-8df0-c01e86c3f23b" />


**Step 4:** Connect to the instance via SSH:

Linux/Unix
```bash name=connect.sh
chmod 400 your-key.pem
ssh -i "your-key.pem" ec2-user@<public-ip-address>
```

PowerShell
```
icacls .\your-key.pem /inheritance:r
icacls .\your-key.pem /grant:r "$env:USERNAME:R"
```

<img width="979" height="122" alt="Screenshot 2026-04-12 at 10 37 32 AM" src="https://github.com/user-attachments/assets/56654d6d-123b-4951-b3b0-b116d2edd018" />

<img width="976" height="277" alt="Screenshot 2026-04-12 at 10 38 00 AM" src="https://github.com/user-attachments/assets/dcafbe9c-1284-4495-8dbc-1f68af39c3c6" />


**Step 5:** Install a web server to demonstrate full control over the instance:

> Note: Before running the command below, make sure to replace the Full Name and Class Roll Number with your own details [Mandatory].

```bash name=setup-httpd.sh
sudo yum install httpd -y
```
```bash name=setup-httpd.sh
sudo systemctl start httpd
```
```bash name=setup-httpd.sh
sudo systemctl enable httpd
```
```bash name=setup-httpd.sh
echo "<html>
<head><title>EC2 Page</title></head>
<body>
<h1>Hello from IaaS - EC2 Instance</h1>
<p><strong>Full Name:</strong> Your Full Name</p>
<p><strong>Class Roll No:</strong> Your Roll Number</p>
</body>
</html>" | sudo tee /var/www/html/index.html
```

> *Screenshot: Browser showing the web page served from the EC2 public IP [Mandatory]*

> Sample:

> <img width="1470" height="302" alt="Screenshot 2026-04-12 at 10 42 16 AM" src="https://github.com/user-attachments/assets/44083df4-8b77-4cab-9f55-8623fa6f8985" />

---

**Part B: PaaS-Deploying with AWS Lambda**

**Step 6:** Navigate to AWS Lambda and click "Create Function". Configure the following:
- Option: `Author from scratch`
- Function name: `PaaS-Lab-Function`
- Runtime: `Python 3.14`
- Architecture: `x86_64`
- Execution role: `Use an existing role` -> `LabRole`

> *Screenshot: Lambda function creation page with settings filled in [Mandatory]*

> Sample:

<img width="1455" height="713" alt="Screenshot 2026-04-12 at 12 13 26 PM" src="https://github.com/user-attachments/assets/df169661-8fd9-4417-8f7d-21c7d6eeef3d" />


**Step 7:** Click "Create Function" and scroll down to the **Code source** section. Replace the default code with the following and click `Deploy`:

```python name=lambda_function.py
import json

def calculate_grade(average):
    if average >= 90:
        return "A"
    elif average >= 80:
        return "B"
    elif average >= 70:
        return "C"
    elif average >= 60:
        return "D"
    else:
        return "F"

def lambda_handler(event, context):
    student_name = event.get("student_name", "Unknown Student")
    scores = event.get("scores", [])

    if not scores:
        return {
            "statusCode": 400,
            "body": json.dumps({"error": "No scores provided."})
        }

    average = sum(scores) / len(scores)
    grade = calculate_grade(average)
    status = "PASS" if average >= 60 else "FAIL"
    highest = max(scores)
    lowest = min(scores)

    result = {
        "student_name": student_name,
        "scores": scores,
        "average": round(average, 2),
        "highest_score": highest,
        "lowest_score": lowest,
        "grade": grade,
        "status": status,
        "remarks": f"{student_name} has {'passed' if status == 'PASS' else 'failed'} with grade {grade}."
    }

    return {
        "statusCode": 200,
        "body": json.dumps(result, indent=2)
    }
```

> *Screenshot: Lambda code editor showing deployed function [Mandatory]*

> Sample:

<img width="1344" height="557" alt="Screenshot 2026-04-12 at 12 16 47 PM" src="https://github.com/user-attachments/assets/10ae1904-adf7-4735-aab5-a1bb1c96d95e" />


**Step 8:** Click `Test` -> `Create new test event`. Configure the following test events and run each one:

- **Test Event 1** Passing Student:
  - Event name: `PassingStudent`
  - Input:
    ```json name=passing_student.json
    {
      "student_name": "Madan Bahadur",
      "scores": [92, 88, 95, 79, 84]
    }
    ```

- **Test Event 2** Failing Student:
  - Event name: `FailingStudent`
  - Input:
    ```json name=failing_student.json
    {
      "student_name": "Hari Bahadur",
      "scores": [45, 52, 38, 60, 41]
    }
    ```

- **Test Event 3** Invalid Input:
  - Event name: `InvalidInput`
  - Input:
    ```json name=invalid_input.json
    {
      "student_name": "Charlie Brown",
      "scores": []
    }
    ```

> *Screenshot: Execution result for each test event showing correct output [Mandatory]*

> Sample:

<img width="1283" height="631" alt="Screenshot 2026-04-12 at 12 21 59 PM" src="https://github.com/user-attachments/assets/3115f923-61a7-4ad5-bb47-4c5d781ea379" />

<img width="1282" height="558" alt="Screenshot 2026-04-12 at 12 21 42 PM" src="https://github.com/user-attachments/assets/9d6be97d-8446-41b5-b608-4f0f9b99ae45" />

<img width="905" height="540" alt="Screenshot 2026-04-12 at 12 24 59 PM" src="https://github.com/user-attachments/assets/ba59b6a6-d82f-4b20-9a3f-dab6d2fbb7dd" />


**Step 9:** Click the `Monitor` tab to view auto-generated CloudWatch metrics. Observe that Lambda automatically tracked **Invocations**, **Duration**, **Error count**, and **Throttles** with no configuration, demonstrating that AWS manages the platform layer entirely.

> *Screenshot: Monitor tab showing invocation metrics for all test runs [Mandatory]*

> Sample:

<img width="1461" height="678" alt="Screenshot 2026-04-12 at 12 25 51 PM" src="https://github.com/user-attachments/assets/0e36b32b-0a8a-4609-9391-ef3ecad1a08e" />

---

**Part C: SaaS-Accessing Amazon WorkDocs**

**Step 10:** Navigate to Amazon WorkDocs and create a new site (or use Amazon Chime as an alternative SaaS example).
  - Register with your email and create a site name
  - Upload a document and share it with another user

*Screenshot: Amazon WorkDocs dashboard showing an uploaded and shared document [Optional]*

---

**Results:**
  - Successfully launched and configured an EC2 instance (IaaS) with a custom web server
  - Deployed a serverless function using AWS Lambda (PaaS) with no infrastructure management required
  - Accessed Amazon WorkDocs (SaaS) as a fully managed service
  - Observed the different levels of management responsibility across all three service models
  - Compare the three models across the following aspects:

| Aspect | IaaS (EC2) | PaaS (Lambda) | SaaS (WorkDocs) |
|---|---|---|---|
| User manages OS | Yes | No | No |
| User manages App | Yes | Yes | No |
| Scalability | Manual/Configured | Automatic | Automatic |
| Flexibility | Highest | Medium | Lowest |
| Setup complexity | High | Medium | Low |


---

**Discussion and Conclusion:**

This lab demonstrated the three fundamental cloud service models in practice. EC2 (IaaS) provided full control over the virtual machine, requiring manual OS configuration and software installation. AWS Lambda (PaaS) abstracted the infrastructure layer completely; only the application code was supplied, and the platform handled provisioning, scaling, and execution automatically, as confirmed by the CloudWatch metrics in the Monitor tab. WorkDocs (SaaS) required no technical setup at all. This progression reflects the core trade off between control and convenience. Organizations select the appropriate model based on their needs: IaaS for maximum flexibility, PaaS for faster development cycles, and SaaS for ready to use solutions.

---

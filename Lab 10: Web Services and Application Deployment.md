# Deploying Web Services on AWS Using Elastic Beanstalk and API Based Architecture

---

**Objectives**
  - Deploy a RESTful web service on AWS Elastic Beanstalk
  - Understand web service concepts, including REST, APIs, and endpoints
  - Configure environment variables and application settings
  - Implement monitoring and logging for web applications
  - Explore AWS App Runner as an alternative deployment platform

---

**Theory**

Web services are software systems designed to support interoperable, machine to machine communication over a network. In cloud computing, they form the backbone of SaaS applications and microservices architectures.

**REST (Representational State Transfer)** is an architectural style that uses standard HTTP methods: GET, POST, PUT, and DELETE to interact with resources identified by URLs. RESTful web services are stateless, cacheable, and horizontally scalable by design.


**AWS Deployment Platforms**

**Elastic Beanstalk** is a Platform as a Service (PaaS) offering that supports deploying web applications written in Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker. It handles infrastructure provisioning automatically.

**AWS App Runner** is a fully managed service for deploying containerized web apps and APIs with minimal configuration.

**Amazon ECS and EKS** provide container orchestration for microservices at scale.

**AWS Amplify** is designed for full stack web and mobile application hosting.

These platforms are AWS equivalents to Google App Engine, Microsoft Azure App Service, and Aneka, all of which represent the PaaS layer of cloud computing.

---

**Procedure**

**Step 1: Create the Flask Application Locally**

1. On your local machine, create a project folder and add the following two files.

> Linux/Unix: Open terminal `mkdir project` , `cd project` , `vim application.py`

> Windows: Open Command Prompt ` mkdir project`, `cd project` , `notepad application.py`

```python name=application.py
from flask import Flask, jsonify, request
import os

application = Flask(__name__)

@application.route('/')
def home():
    return jsonify({
        "service": "Cloud Computing Lab - Web Service",
        "platform": "AWS Elastic Beanstalk",
        "environment": os.environ.get('ENVIRONMENT', 'development'),
        "endpoints": ["/", "/api/students", "/api/courses", "/health"]
    })

@application.route('/api/students', methods=['GET'])
def get_students():
    students = [
        {"id": 1, "name": "David", "course": "Cloud Computing"},
        {"id": 2, "name": "Natasha", "course": "Cloud Computing"},
        {"id": 3, "name": "Harry", "course": "Cloud Computing"}
    ]
    return jsonify({"students": students, "count": len(students)})

@application.route('/api/courses', methods=['GET'])
def get_courses():
    courses = [
        {"id": 1, "name": "Cloud Computing", "units": 6},
        {"id": 2, "name": "Cloud Security", "units": 5},
        {"id": 3, "name": "Virtualization", "units": 10}
    ]
    return jsonify({"courses": courses})

@application.route('/health')
def health():
    return jsonify({"status": "healthy"}), 200

if __name__ == '__main__':
    application.run(debug=True, host='0.0.0.0', port=5000)
```

>Linux: `vim requirements.txt`

>Windows: `notepad requirements.txt`

```text name=requirements.txt
flask==3.0.0
```

> *Screenshot: Take a screenshot of the project folder showing both files [Mandatory]*

> Sample:

<img width="884" height="179" alt="Screenshot 2026-04-23 at 1 16 30 PM" src="https://github.com/user-attachments/assets/71453c88-1af0-4ab3-a974-7e7607d821bb" />

**Step 2: Create the Deployment Package**

1. Open a terminal in your project folder and run the following command to create a ZIP archive:

>Linux/Unix
```bash name=package.sh
zip -r cloud-web-service.zip application.py requirements.txt
```

>`ls`

>Windows 10/11
```
tar -a -c -f cloud-web-service.zip application.py requirements.txt
```

>`dir /p`

2. Confirm that `cloud-web-service.zip` was created in the same directory.


**Step 3: Create an Elastic Beanstalk Application**

1. Open the AWS Management Console and navigate to **Elastic Beanstalk**.
2. Click **Create application**.
3. Set the application name to `cloud-web-service`. The environment name will auto-populate (e.g., `cloud-web-service-env`); leave it as is.
4. For the platform, select **Python** and choose the **Python 3.9+** branch. Leave the platform version as the recommended default.
5. Under application code, select **Local file**. Click **Choose file** and upload the `cloud-web-service.zip` file you just created.
6. Additional environment details: `LabRole`
7. Under **Infrastructure**, configure the following:
   - Environment tier: `Web server environment`
   - Auto scaling group -> Environment type: **Single instance**
   - Autoscaling configuration: `On-Demand instance`
   - Compute -> Architecture: `x86_64`; Instance types: leave default
   - Root volume: leave default
8. Under **Additional infrastructure details**, set the following:
   - EC2 instance profile: `LabInstanceProfile`
   - EC2 key pair: `CloudLab`
9. Under **Additional services -> Monitoring**, set health reporting to **Enhanced**.
10. Leave all other settings as they are and click **Create**.
11. After clicking **Create**, Elastic Beanstalk will begin provisioning the environment and deploying your code. This typically takes **5 to 10 minutes**. Monitor the **Events** tab for progress.
    
>*Screenshot: Take a screenshot of the environment configuration review page before clicking Create [Mandatory].*

>Sample:

<img width="1458" height="712" alt="Screenshot 2026-04-26 at 11 14 27 AM" src="https://github.com/user-attachments/assets/9c2f3aae-3f7b-4f31-a721-7bb0c1b22319" />


**Step 4: Deploy the Application**


2. Once deployment is complete, confirm that the environment health status shows **Ok** in green on the Elastic Beanstalk dashboard.
   
>*Screenshot: Take a screenshot of the Elastic Beanstalk dashboard showing the green **Ok** health status.*

>Sample:

<img width="1470" height="539" alt="Screenshot 2026-04-26 at 11 17 16 AM" src="https://github.com/user-attachments/assets/03e24577-5673-4b95-8989-c6d1ff221a72" />


**Step 5: Test All API Endpoints**

1. On the Elastic Beanstalk environment page, copy the environment URL shown at the top (e.g., `http://cloud-web-service-env.eba-xxxx.us-east-1.elasticbeanstalk.com`).

>*Screenshot:[Mandatory]*

<img width="334" height="64" alt="Screenshot 2026-04-26 at 11 20 25 AM" src="https://github.com/user-attachments/assets/040236fa-b94d-4380-8e11-608f3b56fae3" />


3. Open a browser and test the following endpoints by appending each path to the base URL:

  - Endpoint:`/`
  - Example:
  - Expected Response: Root response from the application

>*Screenshot:[Mandatory]*

<img width="1470" height="154" alt="Screenshot 2026-04-26 at 11 21 52 AM" src="https://github.com/user-attachments/assets/f7fbf752-6571-43a5-8449-318236a0773b" />


  - Endpoint:`/api/students`
  - Example: http://cloud-web-service-env-1.eba-3tfcravw.us-east-1.elasticbeanstalk.com/api/students
  - Expected Response: JSON list of students

>*Screenshot:[Mandatory]*

<img width="1470" height="134" alt="Screenshot 2026-04-26 at 11 22 36 AM" src="https://github.com/user-attachments/assets/4f6e6356-e483-4f50-81af-ee156be46a48" />


  - Endpoint:`/api/courses`
  - Example:
  - Expected Response: JSON list of courses

>*Screenshot:[Mandatory]*

<img width="1145" height="101" alt="Screenshot 2026-04-26 at 12 25 25 PM" src="https://github.com/user-attachments/assets/620a964f-fc68-4aeb-8ea5-e90a654b1f1a" />


  - Endpoint:`/health`
  - Example:
  - Expected Response: `{"status": "healthy"}` with HTTP 200 

>*Screenshot:[Mandatory]*

![Screenshot 2026-04-26 at 12 26 53 PM](https://github.com/user-attachments/assets/b59d3e58-b7cb-4c28-9650-40f9c092ea6d)


**Step 6: View Application Logs**

1. In the Elastic Beanstalk console, open your environment and navigate to the **Logs** section in the left sidebar.
2. Click **Request logs** and select **Last 100 lines**.
3. Once the logs are generated, click **Download** and review the output to confirm that your API requests were handled and recorded.
   
>*Screenshot: Take a screenshot of the log output showing the recorded API requests [Mandatory].*

<img width="1453" height="618" alt="Screenshot 2026-04-26 at 12 33 59 PM" src="https://github.com/user-attachments/assets/da3d28f6-93ef-4cb6-b8f0-d718f06ecba1" />

<img width="1455" height="732" alt="Screenshot 2026-04-26 at 12 34 17 PM" src="https://github.com/user-attachments/assets/bc948786-bae0-4095-a776-6dea1ef0f63c" />

<img width="1458" height="758" alt="Screenshot 2026-04-26 at 12 35 43 PM" src="https://github.com/user-attachments/assets/3f1b2a87-c697-4f81-b254-d102ecf0c0d8" />


**Step 7: Explore AWS App Runner [Optional]**

1. Navigate to the **AWS App Runner** console from the AWS Management Console and click **Create an App Runner service**.
2. Review the available source options:
   - **Container registry** (pulls from Amazon ECR or a public registry)
   - **Source code repository** (connects to GitHub/Bitbucket via a connector)
3. Browse through the configuration options and note the following differences compared to Elastic Beanstalk:

   | Feature | Elastic Beanstalk | App Runner |
   |---|---|---|
   | Infrastructure management | You configure EC2, scaling, VPC | Fully abstracted; no server management |
   | Deployment source | ZIP file / S3 | Source code repo or container image |
   | Auto scaling | Configurable | Automatic, built in |
   | Health checks | Enhanced monitoring via CloudWatch | Built in HTTP health checks |
   | Custom domains | Manual setup | Supported natively |



**Step 8: Set Up a CloudWatch Alarm [Optional]**

  1. Navigate to the **CloudWatch** console and go to **Alarms**.
  2. Click **Create alarm** and select a **metric**.
  3. Choose the **ElasticBeanstalk** metric for `ApplicationRequests5xx`, set the threshold to greater than 5 over 5 minutes.
  4. Configure an action to send a notification via an SNS topic (create a new topic if needed and enter your email address to receive alerts).
  5. Name the alarm and complete the setup.
  6. Take a screenshot of the alarm configuration.

**Step 9: Update the Application [Optional]**

  1. Open `application.py` locally and add a new endpoint. For example:

```python name=application.py
@application.route('/api/info', methods=['GET'])
def get_info():
    return jsonify({"lab": "Lab 10", "topic": "Web Services"})
```

  2. Repackage the application:

```bash name=repackage.sh
zip -r cloud-web-service-v2.zip application.py requirements.txt
```

  3. In the Elastic Beanstalk console, go to your environment and click **Upload and deploy**.
  4. Upload the new ZIP file and confirm the deployment.
  5. Watch the deployment events to observe the rolling update process.
  6. Take a screenshot of the deployment events showing a successful update.

---

**Results**

  - The RESTful web service was deployed successfully on Elastic Beanstalk, and all endpoints returned correct JSON responses.
  - Environment variables and enhanced monitoring were configured through the Elastic Beanstalk console.
  - Application updates were deployed using the upload and deploy workflow, with Elastic Beanstalk handling the rolling update automatically.
  - A CloudWatch alarm was configured to detect elevated 5xx error rates and send notifications.
  - AWS App Runner was reviewed as a fully managed alternative to Elastic Beanstalk.

---

**Discussion and Conclusion**

This lab demonstrated how to deploy and manage a RESTful web service on AWS using Elastic Beanstalk. As a PaaS offering, Elastic Beanstalk abstracts the underlying infrastructure so developers can concentrate on application code. Provisioning, load balancing, scaling, and health monitoring are all handled by the platform automatically.

The RESTful API design used here follows the same web service standards found across all major cloud platforms. Google App Engine, Microsoft Azure App Service, and Aneka serve the same fundamental purpose as Elastic Beanstalk, each offering a managed environment where developers deploy code without managing servers directly.

Comparing Elastic Beanstalk with App Runner illustrates the range within the PaaS category. Elastic Beanstalk offers more configuration flexibility, while App Runner is more opinionated and fully managed, requiring even less setup. Both are suitable depending on how much control versus simplicity a team needs.

CloudWatch alarms add a proactive monitoring layer that is essential in any production deployment. Combined with application logs, they provide the visibility needed to detect and respond to issues quickly. These same patterns apply whether the service is powering a data processing API for scientific workloads or a consumer-facing web application.

---

# Building Serverless Applications Using AWS Lambda and API Gateway

---

**Objectives**

  - Understand serverless computing as a cloud programming model
  - Create and configure AWS Lambda functions
  - Set up API Gateway to trigger Lambda functions
  - Implement event driven architecture with S3 triggers
  - Compare serverless with traditional server based deployment

---

**Theory**

Serverless computing represents the highest level of abstraction in cloud computing. Developers write only the function logic, and the cloud provider handles all infrastructure provisioning, scaling, and availability. AWS Lambda is the leading serverless compute service.

**Characteristics**

**No server management.** AWS automatically provisions and manages the underlying infrastructure on your behalf.

**Event driven execution.** Functions run in response to events such as HTTP requests, S3 file uploads, DynamoDB changes, or scheduled triggers.

**Automatic scaling.** Each function invocation runs independently, and Lambda scales seamlessly to handle thousands of concurrent executions without any manual configuration.

**Pay per execution pricing.** You are billed only for the number of requests and the compute time consumed, measured in milliseconds.

The serverless model closely parallels the task programming model; each Lambda invocation is an independent, isolated task that the platform schedules and executes. Unlike thread based programming where developers manage concurrency manually, Lambda handles parallelism automatically.

AWS API Gateway complements Lambda by exposing RESTful HTTP endpoints that trigger Lambda functions, making it possible to build complete serverless web backends with no servers to manage.

---

**Procedure**

**Step 1: Create the Lambda Function**

  1. Open the AWS Management Console and navigate to Lambda.
  2. Click **Create function** and select **Author from scratch**.
  3. Enter the function name `HelloCloudFunction`.
  4. Set the runtime to **Python 3.xx**.
  5. Under Additional settings -> General, check the box next to "Custom execution role"
  6. A dropdown will appear; select an existing role `LabRole`
  7. Click **Create function**.
    
> *Screenshot: Take a screenshot of the function creation page [Mandatory].*

> Sample:

<img width="1457" height="726" alt="Screenshot 2026-04-22 at 8 56 32 AM" src="https://github.com/user-attachments/assets/5ee0bb5b-ba7e-4c91-ba57-1bff0ed3604f" />


**Step 2: Write and Deploy the Function Code**

1. In the function's code editor, replace the default code with the following:

```python name=lambda_function.py
import json
import datetime

def lambda_handler(event, context):
    name = "World"
    if event.get('queryStringParameters') and event['queryStringParameters'].get('name'):
        name = event['queryStringParameters']['name']

    current_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    response_body = {
        "message": f"Hello, {name}! Welcome to Cloud Computing.",
        "timestamp": current_time,
        "service": "AWS Lambda (Serverless)",
        "event_received": event
    }

    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps(response_body, indent=2)
    }
```

2. Click **Deploy** to save and publish the function.


**Step 3: Test the Function**

1. Click **Test** and create a new test event named `LambdaCloudLab` with the following JSON payload:

```json name=test_event.json
{
  "queryStringParameters": {
    "name": "CloudStudent"
  }
}
```
2. Save the test event and click **Test** again to run it.
3. Verify that the execution result shows a `statusCode` of 200 and the expected message in the response body.

>*Screenshot: Take a screenshot of the test result [Mandatory].*

>Sample:

<img width="1077" height="327" alt="Screenshot 2026-04-22 at 9 01 53 AM" src="https://github.com/user-attachments/assets/abb85386-f872-4080-92ae-1075e2a4ed5a" />


**Step 4: Create an API Gateway Trigger**

  1. Navigate to the API Gateway console and click **Create API**.
  2. Select **HTTP API** and click **Build**.
  3. Name the API `LambdaCloudLabAPI`.
  4. Add an integration, select **Lambda**, and choose `HelloCloudFunction`.
  5. Configure a route: method **GET**, path `/hello`, Integration target `HelloCloudFunction`.
  6. Set the stage to `$default` with auto-deploy enabled.
     
>*Screenshot: Complete the setup and take a screenshot of the API Gateway configuration showing the Lambda integration [Mandatory].*

>Sample:

<img width="1470" height="438" alt="Screenshot 2026-04-22 at 9 10 02 AM" src="https://github.com/user-attachments/assets/fea3ad6c-c001-49b4-890a-f15adb436746" />

<img width="1469" height="420" alt="Screenshot 2026-04-22 at 9 22 25 AM" src="https://github.com/user-attachments/assets/ea1d50fe-e81f-4e2a-bd39-8dfe26b7bf05" />


**Step 5: Test the API Endpoint**

  1. After the API is created, copy the invoke URL shown on the API Gateway dashboard.
  2. Open a browser and navigate to:`https://<api-id>.execute-api.<region>.amazonaws.com/hello?name=Student` (Replace the placeholders with your actual API ID and region)
  >Example: `https://58vozioesh.execute-api.us-east-1.amazonaws.com/hello?name=Student`
  4. Confirm that the browser displays a JSON response with the greeting message.
  
>*Screenshot: Take a screenshot of the browser output [Mandatory].*

>Sample:

<img width="1470" height="820" alt="Screenshot 2026-04-22 at 9 25 59 AM" src="https://github.com/user-attachments/assets/85f2eb98-00f3-40c9-bf0a-aa984c677c4b" />


**Step 6: Create an S3-Triggered Lambda Function**

  1. In **S3** create a general purpose bucket with a unique name (like:`lambdalabsamriddhicollege`) and leave all other settings default.
  2. In **Lambda**, create a new function named `S3ImageProcessor` using the **Python 3.xx** runtime.
  3. Under Additional settings -> General, check the box next to "Custom execution role"
  4. A dropdown will appear; select an existing role `LabRole`
  5. Click **Create function**.
  6. In the function's configuration, click **Add trigger**, Trigger configuration **S3**, choose your bucket `s3/lambdalabsamriddhicollege`, set the event type to `s3:ObjectCreated:*`, and add a suffix filter of `.jpg`.
  7. Replace the default function code with the following:

```python name=s3_processor.py
import json
import boto3

def lambda_handler(event, context):
    s3_client = boto3.client('s3')

    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        size = record['s3']['object']['size']

        print(f"New file uploaded: {key}")
        print(f"Bucket: {bucket}")
        print(f"File size: {size} bytes")

        copy_source = {'Bucket': bucket, 'Key': key}
        new_key = f"processed/{key.split('/')[-1]}"
        s3_client.copy_object(
            CopySource=copy_source,
            Bucket=bucket,
            Key=new_key
        )
        print(f"Copied to: {new_key}")

    return {
        'statusCode': 200,
        'body': json.dumps('Processing complete')
    }
```

  4. Click **Deploy**.


**Step 7: Test the S3 Trigger**

  1. Open the S3 console, navigate to your bucket, and upload any `.jpg` file.
     
>*Sample:*

<img width="1469" height="469" alt="Screenshot 2026-04-22 at 10 28 19 AM" src="https://github.com/user-attachments/assets/fc7eee44-2175-4ea3-b437-30769802b6c7" />


  2. Return to the `S3ImageProcessor` Lambda function, go to the **Monitor** tab, and click **View CloudWatch Logs**.

>*Sample:*

<img width="1460" height="702" alt="Screenshot 2026-04-22 at 10 34 23 AM" src="https://github.com/user-attachments/assets/336e368c-8610-4f70-8c89-ef00567be00b" />
  
  3. Open the most recent log stream and confirm the function executed, printing the file name, bucket, and size.

>*Sample: [Mandatory]*

<img width="1463" height="719" alt="Screenshot 2026-04-22 at 10 37 26 AM" src="https://github.com/user-attachments/assets/0d4f74f8-e6a5-4471-9c25-6600e2c026ef" />
  
  4. Back in the S3 bucket, navigate to the `processed/` folder and confirm the uploaded file was copied there.

>*Sample: [Mandatory]*

<img width="1470" height="296" alt="Screenshot 2026-04-22 at 10 39 46 AM" src="https://github.com/user-attachments/assets/90809630-ed01-4021-9c69-8046325c68c5" />


**Step 8: Monitor Lambda Performance**

  1. Open the `HelloCloudFunction` and go to the **Monitor** tab.
  2. Review the CloudWatch metrics displayed, including Invocations, Duration, Errors, Throttles, and Concurrent executions.

>*Screenshot: Take a screenshot of the monitoring dashboard [Mandatory].*
>Sample:

<img width="1452" height="641" alt="Screenshot 2026-04-22 at 10 46 30 AM" src="https://github.com/user-attachments/assets/36b29196-d72e-4ae0-8aab-24bdfa15f732" />


---

**Results**

  - The Lambda function was created and tested successfully, returning a dynamic JSON response.
  - The API Gateway endpoint was accessible from a browser and responded correctly with query parameter support.
  - The S3 trigger automatically invoked the image processor function upon file upload and copied the file to the `processed/` folder.
  - CloudWatch logs confirmed successful execution and provided visibility into function behavior.
  - No servers were provisioned or managed at any point during the lab.

---

**Discussion and Conclusion**

This lab demonstrated serverless computing as the highest level of cloud abstraction available to developers. Lambda functions embody the task programming model: each invocation is a stateless, independent unit of work that the platform schedules and executes on demand. Developers do not manage threads or concurrency; Lambda handles parallelism automatically across invocations.

API Gateway eliminates the need to run or maintain a web server, enabling a fully serverless HTTP backend. The S3 trigger showcased event driven architecture, a foundational pattern in modern cloud application design where infrastructure responds automatically to data changes without polling or manual orchestration.

Serverless offers real advantages: zero server management overhead, seamless automatic scaling, and a pay per use cost model that eliminates idle resource charges. That said, it comes with trade offs worth understanding. Cold start latency can affect response times for infrequently invoked functions, execution is capped at 15 minutes, and tight integration with AWS specific services introduces a degree of vendor lock in. Evaluating these trade offs against the simplicity and cost benefits is an important part of choosing between serverless and traditional deployment models.

---

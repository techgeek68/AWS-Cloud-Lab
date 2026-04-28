# Automating Cloud Infrastructure Using AWS CloudFormation

---

**Objectives**

 - Understand the principles of Infrastructure as Code (IaC)
 - Create a CloudFormation template to provision AWS resources
 - Deploy a complete multi tier architecture automatically
 - Update and manage stacks using change sets
 - Clean up resources by deleting a stack

---

**Theory**

Infrastructure as Code (IaC) is the practice of defining and managing cloud resources through configuration files rather than performing manual actions in a console. This approach provides four core benefits:

**Repeatability.** The same infrastructure is deployed identically every time, eliminating configuration drift between environments.

**Version control.** Templates are stored in Git, giving teams a full history of infrastructure changes and the ability to roll back when needed.

**Automation.** Entire environments can be provisioned in minutes without human intervention.

**Documentation.** The template itself describes exactly what infrastructure exists and how it is configured, serving as living documentation.

AWS CloudFormation uses YAML or JSON templates to define AWS resources within a stack, a collection of resources managed as a single unit. This aligns with the Service Oriented Architecture (SOA) principles from Unit 2, in which infrastructure components are treated as modular, reusable services with well defined properties and dependencies.

CloudFormation supports all deployment models (public, private, and hybrid) and can provision complex architectures covering VPCs, EC2, RDS, S3, Lambda, and more. It is a foundational tool in the cloud adoption process, enabling organizations to codify their infrastructure for agile deployment and governance.

---

**Procedure**

**Step 1: Create the CloudFormation Template**

- Create a file named `multi-tier-stack.yaml` with the following content:
>Linux: `vim multi-tier-stack.yaml`
>Windows: `cd %USERPROFILE%\Desktop``notepad multi-tier-stack.yaml`

```yaml name=multi-tier-stack.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Cloud Computing Lab - Multi-Tier Web Application Stack'

Parameters:
  EnvironmentName:
    Type: String
    Default: cloud-lab
    Description: Environment name prefix for resources

  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t3.micro
    Description: EC2 instance type

  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of existing EC2 key pair

Resources:
  # VPC
  LabVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub '${EnvironmentName}-vpc'

  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${EnvironmentName}-igw'

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref LabVPC
      InternetGatewayId: !Ref InternetGateway

  # Public Subnet
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref LabVPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: !Sub '${EnvironmentName}-public-subnet'

  # Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref LabVPC
      Tags:
        - Key: Name
          Value: !Sub '${EnvironmentName}-public-rt'

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  # Security Group
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      VpcId: !Ref LabVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: !Sub '${EnvironmentName}-web-sg'

  # EC2 Instance
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyPairName
      ImageId: !Sub '{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64}}'
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install httpd -y
          systemctl start httpd
          systemctl enable httpd
          echo "<h1>Deployed via CloudFormation!</h1><p>Stack: cloud-lab</p>" > /var/www/html/index.html
      Tags:
        - Key: Name
          Value: !Sub '${EnvironmentName}-web-server'

  # S3 Bucket
  LabS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${EnvironmentName}-storage-${AWS::AccountId}'
      VersioningConfiguration:
        Status: Enabled
      Tags:
        - Key: Name
          Value: !Sub '${EnvironmentName}-s3'

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref LabVPC

  WebServerPublicIP:
    Description: Public IP of Web Server
    Value: !GetAtt WebServerInstance.PublicIp

  WebServerURL:
    Description: URL of the Web Server
    Value: !Sub 'http://${WebServerInstance.PublicIp}'

  S3BucketName:
    Description: S3 Bucket Name
    Value: !Ref LabS3Bucket
```

**Step 2: Open the CloudFormation Console and Upload the Template**

  1. In the AWS Management Console, go to **CloudFormation**.
  2. Click **Create stack** and select **With new resources (standard)**.
  3. **Prepare template** > **Choose an existing template** Under **Specify template**, choose **Upload a template file**.
  4. Upload `multi-tier-stack.yaml` and click **Next**.

**Step 3: Configure the Stack Parameters**

 1. Enter `cloud-lab-stack` as the stack name.
 2. Fill in the parameters:
   - **EnvironmentName**: `cloud-lab` (or your preferred prefix)
   - **InstanceType**: `t2.micro`
   - **KeyPairName**: Select an existing EC2 key pair from the dropdown
 3. Click **Next**.
 4. Configure stack options: Leave all defaults.

**Step 4: Review and Create the Stack**

  1. On the review page, check all settings to confirm they are correct.
  2. If prompted, check the box to acknowledge that CloudFormation may create IAM resources.
  3. Click **Submit**.

**Step 5: Monitor Stack Creation**

  1. On the stack detail page, open the **Events** tab.
  2. Watch as CloudFormation creates each resource in dependency order.
  3. Wait until the stack status changes from `CREATE_IN_PROGRESS` to `CREATE_COMPLETE`. This typically takes 3 to 5 minutes.

>*Screenshot: Take a screenshot of the Events tab showing the resource creation progress [Mandatory].*

<img width="1470" height="722" alt="Screenshot 2026-04-28 at 9 46 26 AM" src="https://github.com/user-attachments/assets/f57e5d6f-7742-46fd-90bc-1942bfe4cc59" />


**Step 6: Verify the Deployed Resources**

  1. Go to the **Outputs** tab of the stack and note the value of `WebServerURL`.
  2. Open that URL in a browser. You should see the page displaying "Deployed via CloudFormation!"
  3. Navigate to the EC2, VPC, and S3 consoles to confirm that all resources were created successfully.

>*Screenshot: Take a screenshot of the Outputs tab and a screenshot of the browser showing the web server page [Mandatory].*

<img width="1470" height="253" alt="Screenshot 2026-04-28 at 11 37 25 AM" src="https://github.com/user-attachments/assets/b4f32c25-3e80-4122-adf8-874456f8784f" />


**Step 7: Explore the Template and Resources Tabs**

 1. Open the **Template** tab to view the full YAML template as CloudFormation stored it.
 2. Open the **Resources** tab to see every resource in the stack along with its physical ID and current status.

>*Screenshot: Take a screenshot of the Resources tab [Mandatory].*

<img width="1469" height="542" alt="Screenshot 2026-04-28 at 11 40 16 AM" src="https://github.com/user-attachments/assets/d66a5b5f-1c32-4b96-8170-ea9eb611c2a5" />

<img width="1470" height="361" alt="Screenshot 2026-04-28 at 11 41 53 AM" src="https://github.com/user-attachments/assets/a46144d0-d08c-4a82-9771-39df11e2f41b" />


**Step 8: Update the Stack Using a Change Set**

 1. Open `multi-tier-stack.yaml` and change the `Default` value of `InstanceType` from `t2.micro` to `t3.micro`.
 2. In the **CloudFormation** console, select your stack and click **Update stack**. > **Create a change set**
 3. Choose **Replace existing template** and **Upload a template file** upload the modified file.
 4. InstanceType field — it currently shows `t2.micro`. Manually type `t3.micro`
 5. On the next screen, select **Create change set** instead of executing directly.
 6. Review the proposed changes to confirm only the instance type is being modified.
 7. Click **Execute change set** to apply the update.

>*Screenshot: Take a screenshot of the change set showing the proposed modifications [Mandatory].*

<img width="1470" height="462" alt="Screenshot 2026-04-28 at 12 37 01 PM" src="https://github.com/user-attachments/assets/1ea64b5a-a536-4a8b-a400-806cc72e2d8d" />

<img width="1470" height="572" alt="Screenshot 2026-04-28 at 12 39 53 PM" src="https://github.com/user-attachments/assets/af7e8d85-4672-402f-ac0e-cc853ff3c873" />


**Step 9: Run Drift Detection**

  1. In the CloudFormation console, select your stack.
  2. Click **Stack actions** and choose **Detect drift**.
  3. Wait for the detection to complete, **Stack actions**, then click **View drift results** to review whether any resources were modified outside of CloudFormation.

>*Screenshot: Take a screenshot of the drift detection results [Mandatory].*

<img width="1470" height="646" alt="Screenshot 2026-04-28 at 12 49 00 PM" src="https://github.com/user-attachments/assets/e54e43b4-be18-40e2-8eb2-2813e05655e8" />


**Step 10: Delete the Stack [Optional]**

  1. In the CloudFormation console, select your stack.
  2. Click **Delete** and confirm when prompted.
  3. Monitor the **Events** tab and wait for all resources to be removed.
  4. The stack is fully cleaned up once the status reaches `DELETE_COMPLETE`. All provisioned resources, including the VPC, EC2 instance, security groups, and S3 bucket, are removed automatically.

>*Screenshot: Take a screenshot of the Events tab showing the deletion in progress.*

---

**Results**

  - The CloudFormation template successfully defined a multi tier architecture covering a VPC, public subnet, EC2 web server, and S3 bucket.
  - The stack provisioned all resources automatically in approximately 3 to 5 minutes.
  - The web server was accessible via the output URL and displayed the expected page.
  - A change set demonstrated controlled, reviewable infrastructure updates.
  - Stack deletion cleanly removed all provisioned resources with no manual cleanup required.

---

**Discussion and Conclusion**

This lab demonstrated Infrastructure as Code in practice using AWS CloudFormation. By defining all resources in a single YAML template, an entire cloud architecture spanning networking, compute, and storage was provisioned in a single operation, with no manual console work.

This approach reflects SOA principles: each resource is a modular component with clearly defined properties and dependencies. The Parameters section makes the template reusable across different environments such as development, staging, and production, while the Outputs section exposes key values programmatically for downstream use.

Change sets add a critical safety layer for production environments by allowing teams to preview exactly what will change before any modification is applied. Drift detection complements this by identifying resources that were changed outside of CloudFormation, helping maintain consistency between the declared template and the actual infrastructure state.

IaC is a cornerstone of modern cloud adoption. It accelerates deployment, reduces human error, enables version controlled infrastructure history, and provides the consistency and repeatability that dynamic cloud environments require.

---

# Implementing Cloud Architectures using AWS VPC, Public/Private Subnets, and Hybrid Connectivity

**Objectives:**
  * Design and implement a Virtual Private Cloud (VPC) simulating cloud deployment models.
  * Configure public and private subnets to represent public and private cloud zones.
  * Set up an Internet Gateway, NAT Gateway, and route tables.
  * Understand hybrid cloud architecture concepts using VPN or peering.
  * Implement security groups and network ACLs for enhanced security.

---

**Theory**

Cloud deployment models determine the infrastructure's location and who manages it:  
  - **Public Cloud:**
    - Shared resources available over the internet (e.g., AWS, Azure). It is cost effective with a pay per use billing approach.  
  - **Private Cloud:**
    - Dedicated resources for a single organization offering higher control, security, and compliance. On AWS, a VPC with private subnets simulates a private cloud.  
  - **Community Cloud:**
    - Shared by organizations with similar concerns (e.g., healthcare, government).  
  - **Hybrid Cloud:**
    - Combines public and private clouds, enabling data and applications to move between environments. AWS supports hybrid setups via VPN, Direct Connect, and AWS Outposts.

Amazon VPC (Virtual Private Cloud) allows you to create a logically isolated section of AWS with custom IP ranges, subnets, route tables, and gateways. Service Oriented Architecture (SOA) principles also apply, ensuring cloud services interact through defined APIs.

---

**Procedure**

**Step 1: Create a VPC**  
- Go to the AWS Management Console and open the VPC service. **VPC → Create VPC**.  
- Select `VPC and more`. 
   - **Name tag:** cloud-arch-lab  
   - **IPv4 CIDR block:** 10.0.0.0/16  
   - **Availability Zones:** 2  
   - **Public subnets:** 2 (10.0.1.0/24, 10.0.2.0/24)  
   - **Private subnets:** 2 (10.0.3.0/24, 10.0.4.0/24)
   - **Customize subnets CIDR blocks**
     - Public subnet CIDR block in us-east-1a: 10.0.1.0/24
     - Public subnet CIDR block in us-east-1b: 10.0.2.0/24
     - Private subnet CIDR block in us-east-1a: 10.0.3.0/24
     - Private subnet CIDR block in us-east-1b: 10.0.4.0/24
   - **NAT gateways:** `Zonal`: In 1 AZ (select 1 per AZ or In 1 AZ to reduce costs)
   - **VPC endpoints:** S3 Gateway
   - **Create VPC**

>*Screenshot: The page that shows the details of the VPC you created [Mandatory]*

>Sample:

<img width="1469" height="773" alt="Screenshot 2026-04-15 at 8 09 58 AM" src="https://github.com/user-attachments/assets/e8935c8b-b1c1-48bb-bd69-8845cc2e0613" />


**Step 2: Review the VPC topology.**  
Check the auto generated resource map displaying the public and private subnets, Internet Gateway, and NAT Gateway.

>*Screenshot: VPC resource map showing public/private subnets, IGW, and NAT GW. [Mandatory]*

>Sample:

<img width="1259" height="396" alt="Screenshot 2026-04-15 at 8 10 32 AM" src="https://github.com/user-attachments/assets/97050c0c-6af8-4cdc-902a-90de6ce32205" />

<img width="1258" height="447" alt="Screenshot 2026-04-15 at 8 10 45 AM" src="https://github.com/user-attachments/assets/72f44baf-4641-45d2-8076-ed234a5a0292" />

<img width="1255" height="439" alt="Screenshot 2026-04-15 at 8 10 56 AM" src="https://github.com/user-attachments/assets/9c77ad59-e2fa-4c58-adff-4a7baa1711d2" />

<img width="1263" height="443" alt="Screenshot 2026-04-15 at 8 11 06 AM" src="https://github.com/user-attachments/assets/e3475686-0037-4fe9-9fe3-7e678d898a31" />


**Step 3: Verify Route Tables.**  
  - **Public route table:** Traffic (0.0.0.0/0) directed to the Internet Gateway, simulating public cloud access.
    
>Screenshot: Route table entries for public subnets [Mandatory].

>Sample:

<img width="1458" height="544" alt="Screenshot 2026-04-15 at 8 14 44 AM" src="https://github.com/user-attachments/assets/4da0e9ed-1f6e-4625-888a-3fb250d86edd" />


  - **Private route table:** Traffic (0.0.0.0/0) directed to the NAT Gateway, ensuring controlled outbound access.

>Screenshot: Route table entries for private subnets [Mandatory].

>Sample:

<img width="1459" height="567" alt="Screenshot 2026-04-15 at 8 16 09 AM" src="https://github.com/user-attachments/assets/ef915750-f1e6-4d9b-9047-76e163f19b9e" />

<img width="1459" height="563" alt="Screenshot 2026-04-15 at 8 16 24 AM" src="https://github.com/user-attachments/assets/404b09e2-3a72-4df6-9770-fb61d9bdd4c2" />


**Step 4: Launch an EC2 Instance in the Public Subnet.**  
  - Go to EC2 > Instances > Launch instance.
  - **Name:** `Public WebServer`
    - Application and OS Images (Amazon Machine Image): `Amazon Linux`
    - Instance type: `t2.micro`
    - Select or create (If you don't already have one) a key pair and launch.
  - **Network settings**:
    - VPC: `cloud-arch-lab`
    - Subnet: `cloud-arch-lab-subnet-public1-us-east-1a` (select one of the public subnets)
    - Auto-assign Public IP: `Enable`
    - Security group: Create
      - Name: My Firewall 1
      - Description: `Allow SSH, HTTP, and HTTPS from anywhere (0.0.0.0/0)`
      - Allow SSH (22), HTTP (80), and HTTPS (443) from 0.0.0.0/0
    - Configure storage: 8 GiB, GP3

> Screenshot: EC2 launch configuration showing public subnet selection [Mandatory].

> Sample:

<img width="1459" height="504" alt="Screenshot 2026-04-15 at 8 37 56 AM" src="https://github.com/user-attachments/assets/dd26e14c-098a-41f1-b245-4556f1aa6b2c" />


**Step 5: Launch an EC2 Instance in the Private Subnet.**  
  - Launch instance.
  - **Name:** `Private DBServer`  
    - Application and OS Images (Amazon Machine Image): `Amazon Linux`
    - Instance type: `t2.micro`
    - Select or create (If you don't already have one) a key pair and launch.
  - **Network settings**:
    - VPC: `cloud-arch-lab`
    - Subnet: `cloud-arch-lab-subnet-private1-us-east-1a` (select one of the private subnets)
    - Auto-assign Public IP: `Disable`  
    - Security group: Create
      - Name: `My Firewall 2`
      - Description: `Allow SSH and MySQL only from (10.0.0.0/16)`
      - Allow MySQL (3306) and Allow SSH (22) only from 10.0.0.0/16.
>Sample:
<img width="861" height="539" alt="Screenshot 2026-04-15 at 9 09 05 AM" src="https://github.com/user-attachments/assets/41310c24-73f6-4455-b85e-5aea7aff468f" />

    - Configure storage: 8 GiB, GP3

> Screenshot: EC2 launch configuration showing private subnet [Mandatory].

> Sample:

<img width="1459" height="505" alt="Screenshot 2026-04-15 at 9 12 53 AM" src="https://github.com/user-attachments/assets/d5d5dc0c-7e99-46a4-a5da-e065fe0527c7" />


**Step 6: Test Connectivity.**  
  - **SSH into the public instance** and from there, **SSH into the private instance** (using the private IP).
  - Syntax:
  ```bash
  ssh -i key.pem ec2-user@<public-instance-public-ip>
  ```
  - Example:
  ```
  ssh -i "CloudLab.pem" ec2-user@ec2-44-199-212-187.compute-1.amazonaws.com
  ```
  ```
  ping -c 4 www.google.com
  ```

>Screenshot: Terminal showing a successful SSH hop and a ping from a public instance [Mandatory].

> Sample:

<img width="1049" height="304" alt="Screenshot 2026-04-15 at 9 18 21 AM" src="https://github.com/user-attachments/assets/fa62d653-1c0e-4aa6-9f78-a6a9c1b0a77c" />

<img width="1047" height="164" alt="Screenshot 2026-04-15 at 9 19 14 AM" src="https://github.com/user-attachments/assets/2d151aa5-b4d1-44f8-bc85-2a4ce5a988bd" />

  - From the private instance, SSH from the public instance there, **SSH into the private instance** (using the private IP)
    - Example:
    ```
    chmod 400 ~/Download/CloudLab.pem      #Set Permission
    ```
    ```
    ssh -i "CloudLab.pem" -J ec2-user@44.199.212.187 ec2-user@10.0.3.213
    ```
  - verify outbound internet connectivity through the NAT Gateway:  
   ```bash
   ping google.com
   ```

>Screenshot: Terminal showing a successful SSH hop and a ping from a private instance [Mandatory]. 

>Sample:

<img width="788" height="328" alt="Screenshot 2026-04-15 at 12 19 36 PM" src="https://github.com/user-attachments/assets/6d6a6e94-54a1-48ff-82a0-3dc7ad8db421" />


**Step 7: Configure Network ACLs for Additional Security.**  
  - Navigate to VPC > Network ACLs
  - Select the NACL associated with the private subnets
  - Inbound rules: Edit to Deny all traffic
  - Add Allow rules for traffic from the public subnet CIDRs (e.g., `10.0.1.0/24` and `10.0.2.0/24`)

>Screenshot: Network ACL rules for private subnet.

>Sample:

<img width="1458" height="669" alt="Screenshot 2026-04-15 at 12 35 09 PM" src="https://github.com/user-attachments/assets/0be10d77-c977-459f-a0c1-eb2f9947d33b" />

<img width="1160" height="370" alt="Screenshot 2026-04-15 at 12 36 57 PM" src="https://github.com/user-attachments/assets/5905d988-2c98-4608-ae78-6125307bd546" />


**Step 8: Explore Hybrid Cloud Concepts (Observation Only)**  
  - Navigate to **VPC** > S**ite-to-Site** VPN Connections.
  - Click Create VPN connection (do not complete creation).
  - Observe required fields:
    - **Target Gateway Type**: Virtual Private Gateway (created separately).
    - **Virtual Private Gateway**: Dropdown (empty until resource exists).
    - **Customer Gateway**: Requires on-premises public IP and BGP ASN.
    - **Routing Options**: Static or Dynamic (BGP).
  - Click Cancel to exit without creating resources.
    

**Step 9: Create a VPC Peering Connection (Observation Only)**  
- Navigate to **VPC > Peering Connections**.
- Click **Create peering connection**.
- Configure request:
  - **Name**: Provide a descriptive name (e.g., `vpc-a-to-vpc-b`).
  - **Requester**: Select your current VPC.
  - **Accepter**: Select another VPC in your account/region.
- Click **Create peering connection**.
- Select the new pending request in the list.
- Click **Actions** > **Accept request**.
- In the confirmation dialog, click **Accept request**.
- **Update Route Tables** (repeat for both VPCs):
  - Navigate to **Route Tables**.
  - Select the main route table for **VPC A**.
  - **Routes** tab > **Edit routes** > **Add route**:
    - Destination: CIDR block of **VPC B** (e.g., `10.1.0.0/16`).
    - Target: Select **Peering Connection** and choose the new PCX ID.
  - **Save routes**.
  - Repeat process for **VPC B** route table, targeting **VPC A** CIDR block.

---

**Results**

The lab resulted in a functional VPC with a 10.0.0.0/16 CIDR block spanning two Availability Zones. Public subnets were made accessible via an Internet Gateway, while private subnets maintained outbound internet access through a NAT Gateway. The public web server was reachable from the internet, and the private database server was restricted to internal VPC access only. The configuration options for VPN and VPC Peering demonstrated foundational hybrid cloud connectivity.

---

**Discussion and Conclusion**

This lab illustrated how AWS VPC components map directly to common cloud architecture patterns. Public subnets equipped with an Internet Gateway represent a public cloud model, while private subnets behind a NAT Gateway simulate a secure private cloud deployment. The use of a NAT Gateway allows backend resources to retrieve updates without exposing them to inbound internet connections. VPN connections and VPC Peering are essential tools for building hybrid environments that connect on premises infrastructure with cloud resources. This design follows Service Oriented Architecture principles by isolating the web and database tiers and restricting communication to specific interfaces, which strengthens both security and operational maintainability.

---

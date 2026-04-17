# Implementing Load Balancing and Auto Scaling on AWS

---

**Objectives**

- Configure an Application Load Balancer to distribute incoming traffic.
- Set up an Auto Scaling group using a launch template.
- Implement scaling policies that respond to CloudWatch metrics.
- Test horizontal scaling by simulating increased load.
- Understand the relationship between virtualization, load balancing, and elasticity.

---

**Theory**

Load balancing is the practice of distributing incoming network traffic across multiple target resources, such as EC2 instances or containers. This distribution ensures high availability and fault tolerance by preventing any single resource from becoming overwhelmed. AWS offers several load balancer types tailored to different needs. The Application Load Balancer operates at Layer 7 of the OSI model, handling HTTP and HTTPS traffic and supporting advanced routing based on request path or host headers. The Network Load Balancer operates at Layer 4, providing ultra low latency for TCP and UDP traffic. The Gateway Load Balancer is designed for integrating third party virtual appliances into the network path.

Auto Scaling is the mechanism that automatically adjusts the number of EC2 instances in response to changing demand. During periods of high utilization, Auto Scaling adds instances, a process known as scaling out. When demand subsides, it removes unneeded instances, scaling in, to optimize costs. Scaling actions can be governed by policies such as target tracking, which maintains a specified metric threshold like average CPU utilization, step scaling, or scheduled actions.

This dynamic behavior is a direct benefit of cloud virtualization. The ability to provision and terminate server instances programmatically in seconds makes elastic, resilient architectures possible.

---

**Procedure**

**Step 1: Create Security Groups**

**1.1 Create ALB Security Group:**
- Navigate to **EC2 > Security Groups > Create security group**
- Set Security group name to `web-alb-sg`
- Set Description to `Security group for Application Load Balancer`
- **Inbound rules:**
  - Type: HTTP, Port: 80, Source: 0.0.0.0/0, Description: Allow HTTP from the internet
- **Outbound rules:** (Keep default - All traffic to 0.0.0.0/0)
- Click **Create security group**

**1.2 Create Instance Security Group:**
- Navigate to **EC2 > Security Groups > Create security group**
- Set Security group name to `web-instance-sg`
- Set Description to `Security group for web server instances`
- **Inbound rules:**
  - **Type: HTTP, Port: 80, Source: 0.0.0.0/0, Description: Allow HTTP from anywhere**
  - Type: SSH, Port: 22, Source: My IP, Description: SSH access
- **Outbound rules:** (Keep default - All traffic to 0.0.0.0/0)
- Click **Create security group**


**Step 2: Create the Launch Template**
- Navigate to **EC2 > Launch Templates > Create launch template**
- Set Launch template name to `web-server-template`
- Under **Application and OS Images**, select **Amazon Linux 2023 AMI**
- Under **Instance type**, select `t2.micro`
- Under **Key pair**, select an existing key pair
- Under **Network settings > Security groups**, select `web-instance-sg`
- Expand **Advanced details**
- In the **User data** field, paste this script:

```bash
#!/bin/bash
# Corrected user data script for Amazon Linux 2023

# Enhanced logging
exec > >(tee /var/log/user-data.log) 2>&1
echo "=== User Data Script Started at $(date) ==="

# Wait for the system to be ready
sleep 10

# Update system using dnf (Amazon Linux 2023 uses dnf, not yum)
echo "Updating system packages..."
dnf update -y
echo "System update completed"

# Install httpd
echo "Installing httpd..."
dnf install -y httpd
echo "httpd installation completed"

# Start and enable httpd
echo "Starting httpd service..."
systemctl start httpd
systemctl enable httpd
echo "httpd service started and enabled"

# Wait for metadata service to be available
echo "Waiting for metadata service..."
sleep 5

# Get instance metadata with retry logic
echo "Retrieving instance metadata..."
for i in {1..5}; do
    INSTANCE_ID=$(curl -s --connect-timeout 10 http://169.254.169.254/latest/meta-data/instance-id)
    if [ ! -z "$INSTANCE_ID" ]; then
        break
    fi
    echo "Retry $i for instance ID..."
    sleep 3
done

for i in {1..5}; do
    AZ=$(curl -s --connect-timeout 10 http://169.254.169.254/latest/meta-data/placement/availability-zone)
    if [ ! -z "$AZ" ]; then
        break
    fi
    echo "Retry $i for AZ..."
    sleep 3
done

# Use default values if metadata fails
INSTANCE_ID=${INSTANCE_ID:-"Unknown"}
AZ=${AZ:-"Unknown"}

echo "Instance ID: $INSTANCE_ID"
echo "Availability Zone: $AZ"

# Create HTML content
echo "Creating web page..."
cat > /var/www/html/index.html << EOF
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Load Balancer Test - $INSTANCE_ID</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            text-align: center;
            margin: 0;
            padding: 50px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .container {
            background: rgba(255, 255, 255, 0.1);
            padding: 40px;
            border-radius: 15px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(31, 38, 135, 0.37);
            border: 1px solid rgba(255, 255, 255, 0.18);
        }
        .instance-id {
            color: #FFD700;
            font-size: 28px;
            font-weight: bold;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }
        .az {
            color: #00FF7F;
            font-size: 20px;
            font-weight: bold;
        }
        .timestamp {
            color: #FFB6C1;
            font-style: italic;
        }
        h1 {
            font-size: 36px;
            margin-bottom: 30px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Load Balancer Test</h1>
        <div class="instance-id">Instance: $INSTANCE_ID</div>
        <br>
        <div class="az">Zone: $AZ</div>
        <br>
        <div class="timestamp">Generated: $(date)</div>
        <hr style="border: 1px solid rgba(255,255,255,0.3); margin: 30px 0;">
        <p>Web Server Running Successfully!</p>
        <p>Refresh to test load balancing</p>
    </div>
</body>
</html>
EOF

# Set proper permissions
chown apache:apache /var/www/html/index.html
chmod 644 /var/www/html/index.html

# Ensure httpd is running
systemctl restart httpd

# Verify everything is working
echo "Verifying installation..."
systemctl status httpd --no-pager
curl -s localhost | head -5

echo "=== User Data Script Completed Successfully at $(date) ==="
```
- Click **Create launch template**

> *Screenshot: Launch template with corrected user data script [Mandatory]*

> Sample:

<img width="1457" height="698" alt="Screenshot 2026-04-17 at 6 46 54 AM" src="https://github.com/user-attachments/assets/27b90576-5e9a-4bfd-818d-3f346791d978" />



**Step 3: Create a Target Group**
- Navigate to **EC2 > Load Balancing > Target Groups > Create target group**
- Under **Basic configuration**, choose `Instances` as the target type
- Set Target group name to `web-target-group`
- Set Protocol to `HTTP` and Port to `80`
- Under **Health checks**:
  - Health check protocol: `HTTP`
  - Health check path: `/`
  - **Advanced health check settings:**
    - Healthy threshold: `2`
    - Unhealthy threshold: `3`
    - Timeout: `10 seconds`
    - Interval: `30 seconds`
    - Success codes: `200`
- Click **Next**, then **Create target group**

> *Screenshot: Target group configuration [Mandatory]*

> Sample:

<img width="1458" height="640" alt="Screenshot 2026-04-17 at 6 51 07 AM" src="https://github.com/user-attachments/assets/c1982670-a27e-4cb9-b597-413b2c6757cd" />



**Step 4: Create the Application Load Balancer**
- Navigate to **EC2 > Load Balancers > Create load balancer**
- Select `Application Load Balancer` and click **Create**
- Set Load balancer name to `web-alb`
- Set Scheme to `Internet-facing`
- Set IP address type to `IPv4`
- Under **Network mapping**, select your VPC and choose **at least two public subnets** in different AZs
- Under **Security groups**, **remove the default** and select `web-alb-sg`
- Under **Listeners and routing**:
  - Protocol: HTTP, Port: 80
  - Default action: Forward to `web-target-group`
- Click **Create load balancer**

> *Screenshot: ALB configuration with correct security group [Mandatory]*

> Sample:

<img width="1457" height="557" alt="Screenshot 2026-04-17 at 10 53 50 AM" src="https://github.com/user-attachments/assets/5629ea77-360f-4544-be1f-f3ecf6b64d97" />


**Step 5: Create the Auto Scaling Group**
  - Navigate to **EC2 > Auto Scaling > Auto Scaling Groups > Create Auto Scaling group**
  - Set Auto Scaling group name to `web-asg`
  - Under **Launch template**, select `web-server-template` (latest version) and click **Next**
  - Under **Network**:
    - Select your VPC
    - Choose the **same public subnets** you selected for the ALB
    - Click **Next**
  - Under **Load balancing**:
    - Select `Attach to an existing load balancer`
    - Choose `Choose from your load balancer target groups`
    - Select `web-target-group`
    - **Health checks:** Select both `EC2` and `ELB` health checks
    - **Health check grace period:** `400 seconds` (increased for user data execution)
    - Click **Next**
  - Under **Group size**:
    - Desired capacity: `2`
    - Minimum capacity: `1`
    - Maximum capacity: `4`
  - Under **Scaling policies**:
    - Select `Target tracking scaling policy`
    - Metric type: `Average CPU utilization`
    - Target value: `50`
  - Click **Next** through remaining options, then **Create Auto Scaling group**

> *Screenshot: Auto Scaling Group with increased health check grace period [Mandatory]*

> Sample:

<img width="1456" height="715" alt="Screenshot 2026-04-17 at 11 21 51 AM" src="https://github.com/user-attachments/assets/12d9fd04-668a-4e10-a274-7589279fae2a" />


**Step 6: Verify Initial Instances**
  - **Wait 8-12 minutes** for instances to launch, install software, and pass health checks
  - Navigate to **EC2 > Instances** and confirm two instances with prefix `web-asg` are **Running**
  - **Test individual instances:** Copy public IP of one instance and test `http://[public-ip]` in browser
    
> Sample [Optional]:

<img width="1282" height="833" alt="Screenshot 2026-04-17 at 11 23 02 AM" src="https://github.com/user-attachments/assets/4a555ad3-371f-4879-84f6-834a6a7d0166" />

  - Navigate to **Target Groups > web-target-group > Targets tab** and confirm both instances show **Healthy** status

> *Screenshot: Target group showing healthy instances [Mandatory]*

> Sample:

<img width="1456" height="716" alt="Screenshot 2026-04-17 at 11 24 18 AM" src="https://github.com/user-attachments/assets/222260c8-64de-419e-bca3-5f0757f610fc" />


>*Troubleshooting if instances remain unhealthy (do not include this on lab report):*
>SSH into an instance and check:
>```bash
>sudo systemctl status httpd
>curl localhost
>sudo tail -f /var/log/user-data.log
>```

**Step 7: Test Load Balancing**
  - Navigate to **EC2 > Load Balancers**
  - Select `web-alb` and copy the **DNS name** from the **Description** tab
  - **Wait 2-3 minutes** after ALB shows "Active" status
  - Open browser and navigate to: `http://[your-alb-dns-name]`
  - **Refresh multiple times** - you should see different Instance IDs and Availability Zones
  - The page should display the beautiful gradient design with instance information

> *Screenshot: Browser showing different instance IDs on consecutive refreshes [Mandatory]*

> Sample:

<img width="1075" height="795" alt="Screenshot 2026-04-17 at 11 29 44 AM" src="https://github.com/user-attachments/assets/a4b13daf-dedd-48d8-befa-f01974535608" />


**Step 8: Trigger a Scale Out Event**
  - SSH into one of the running instances:
  ```bash
  ssh -i your-key.pem ec2-user@[instance-public-ip]
  ```
  - Install stress tool:
  ```bash
  sudo dnf install -y stress
  ```
  - Install and run stress tool:
  ```bash
  stress --cpu 4 --timeout 300s
  ```
  - Navigate to **Auto Scaling Groups > web-asg > Activity tab**
    
> Sample:

<img width="1457" height="717" alt="Screenshot 2026-04-17 at 11 41 44 AM" src="https://github.com/user-attachments/assets/bcc2c250-6c99-45f0-b7d4-931acd0db366" />

  - Within 3-5 minutes, observe the new instance launching
  - Monitor **CloudWatch > Alarms** to see CPU alarm trigger
    
> Sample:

<img width="1459" height="346" alt="Screenshot 2026-04-17 at 11 53 26 AM" src="https://github.com/user-attachments/assets/86ea0cc3-8a42-47ad-8d5f-787e1e5e1caa" />


**Step 9: Observe Scale In Behavior**
  - Allow stress test to complete (5 minutes)
  - Return to **Auto Scaling Groups > web-asg > Activity tab**
  - After the cooldown period (5-10 minutes), observe the instance termination
  - Verify desired capacity returns to 2


**Step 10: Test Self Healing**
  - Navigate to **EC2 > Instances**
  - Select one Auto Scaling managed instance
  - **Instance state > Terminate instance > Terminate**
  - Check **Auto Scaling Groups > web-asg > Activity tab**
  - Confirm new instance launches automatically within 2-3 minutes

---

**Results**

The Application Load Balancer successfully distributed web traffic across multiple EC2 instances. The Auto Scaling group maintained the configured desired capacity and automatically scaled out when the simulated CPU load exceeded the 50 percent target threshold. After the load subsided, the group scaled back in, demonstrating cost efficiency. The system also demonstrated self healing behavior by automatically replacing a manually terminated instance.

---

**Discussion and Conclusion**

This lab illustrated two fundamental capabilities of cloud infrastructure that are enabled by virtualization. The Application Load Balancer provided a single point of entry while distributing requests across a pool of instances, eliminating single points of failure and improving responsiveness. Auto Scaling delivered horizontal elasticity by dynamically matching the number of running instances to real time demand. The self healing nature of the Auto Scaling group further enhanced system resilience. The combination of load balancing and auto scaling forms the foundation for designing highly available, scalable, and cost effective applications in the cloud.

---

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

exec > >(tee /var/log/user-data.log) 2>&1
echo "=== User Data Script Started at $(date) ==="

sleep 10

echo "Updating system packages..."
dnf update -y
echo "System update completed"

echo "Installing httpd..."
dnf install -y httpd
echo "httpd installation completed"

echo "Starting httpd service..."
systemctl start httpd
systemctl enable httpd
echo "httpd service started and enabled"

echo "Waiting for metadata service..."
sleep 5

# -------------------------------------------------------
# IMDSv2: get a session token first, then use it for all
# metadata requests (required on Amazon Linux 2023 by default)
# -------------------------------------------------------
echo "Retrieving IMDSv2 token..."
for i in {1..5}; do
    TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
        -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" \
        --connect-timeout 10)
    if [ -n "$TOKEN" ]; then
        echo "IMDSv2 token acquired"
        break
    fi
    echo "Retry $i for IMDSv2 token..."
    sleep 3
done

echo "Retrieving instance metadata..."
for i in {1..5}; do
    INSTANCE_ID=$(curl -s --connect-timeout 10 \
        -H "X-aws-ec2-metadata-token: $TOKEN" \
        http://169.254.169.254/latest/meta-data/instance-id)
    if [ -n "$INSTANCE_ID" ]; then break; fi
    echo "Retry $i for instance ID..."
    sleep 3
done

for i in {1..5}; do
    AZ=$(curl -s --connect-timeout 10 \
        -H "X-aws-ec2-metadata-token: $TOKEN" \
        http://169.254.169.254/latest/meta-data/placement/availability-zone)
    if [ -n "$AZ" ]; then break; fi
    echo "Retry $i for AZ..."
    sleep 3
done

INSTANCE_ID=${INSTANCE_ID:-"Unavailable"}
AZ=${AZ:-"Unavailable"}

echo "Instance ID: $INSTANCE_ID"
echo "Availability Zone: $AZ"

echo "Creating web page..."
cat > /var/www/html/index.html << EOF
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Load Balancer Test — $INSTANCE_ID</title>
    <style>
        *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Inter', 'Segoe UI', sans-serif;
            background-color: #0f1117;
            color: #e2e8f0;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 2rem;
        }

        .card {
            background: #1a1d27;
            border: 1px solid #2d3148;
            border-radius: 12px;
            padding: 2.5rem 3rem;
            width: 100%;
            max-width: 480px;
            box-shadow: 0 4px 24px rgba(0, 0, 0, 0.4);
        }

        .card-header {
            margin-bottom: 2rem;
            padding-bottom: 1.25rem;
            border-bottom: 1px solid #2d3148;
        }

        .badge {
            display: inline-block;
            font-size: 0.7rem;
            font-weight: 600;
            letter-spacing: 0.08em;
            text-transform: uppercase;
            color: #6c8ebf;
            background: #1e2640;
            border: 1px solid #2d3a5c;
            border-radius: 4px;
            padding: 3px 8px;
            margin-bottom: 0.75rem;
        }

        h1 {
            font-size: 1.5rem;
            font-weight: 600;
            color: #f1f5f9;
            letter-spacing: -0.01em;
            line-height: 1.3;
        }

        .meta-grid {
            display: grid;
            gap: 0.85rem;
            margin-bottom: 2rem;
        }

        .meta-row {
            display: flex;
            justify-content: space-between;
            align-items: baseline;
            gap: 1rem;
        }

        .meta-label {
            font-size: 0.8rem;
            color: #64748b;
            font-weight: 500;
            white-space: nowrap;
            flex-shrink: 0;
        }

        .meta-value {
            font-size: 0.875rem;
            font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
            color: #94b8ff;
            font-weight: 500;
            text-align: right;
            word-break: break-all;
        }

        .meta-value.zone {
            color: #7ecfb3;
        }

        .meta-value.time {
            color: #9ba8bb;
            font-size: 0.8rem;
        }

        .divider {
            border: none;
            border-top: 1px solid #2d3148;
            margin: 0 0 1.5rem;
        }

        .status {
            display: flex;
            align-items: center;
            gap: 0.5rem;
            font-size: 0.82rem;
            color: #7ecfb3;
            font-weight: 500;
            margin-bottom: 0.5rem;
        }

        .status::before {
            content: '';
            display: inline-block;
            width: 7px;
            height: 7px;
            border-radius: 50%;
            background: #7ecfb3;
            flex-shrink: 0;
        }

        .hint {
            font-size: 0.78rem;
            color: #475569;
            padding-left: 1.1rem;
        }
    </style>
</head>
<body>
    <div class="card">
        <div class="card-header">
            <div class="badge">EC2 · Load Balancer</div>
            <h1>Instance Info</h1>
        </div>

        <div class="meta-grid">
            <div class="meta-row">
                <span class="meta-label">Instance ID</span>
                <span class="meta-value">$INSTANCE_ID</span>
            </div>
            <div class="meta-row">
                <span class="meta-label">Availability Zone</span>
                <span class="meta-value zone">$AZ</span>
            </div>
            <div class="meta-row">
                <span class="meta-label">Generated</span>
                <span class="meta-value time">$(date -u '+%Y-%m-%d %H:%M UTC')</span>
            </div>
        </div>

        <hr class="divider">

        <div class="status">Server running</div>
        <div class="hint">Refresh the page to verify load balancing across instances.</div>
    </div>
</body>
</html>
EOF

chown apache:apache /var/www/html/index.html
chmod 644 /var/www/html/index.html

systemctl restart httpd

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

<img width="1457" height="533" alt="Screenshot 2026-04-19 at 7 18 07 AM" src="https://github.com/user-attachments/assets/963023f7-e4f5-4443-92fd-12b5839b99fe" />



**Step 6: Verify Initial Instances**
  - **Wait 8-12 minutes** for instances to launch, install software, and pass health checks
  - Navigate to **EC2 > Instances** and confirm two instances with prefix `web-asg` are **Running**
    
> Sample :

<img width="1444" height="311" alt="Screenshot 2026-04-19 at 7 17 03 AM" src="https://github.com/user-attachments/assets/05992dd3-5cc1-4518-8f0d-3a28572b9f04" />


  - **Test individual instances:** Copy public IP of one instance and test `http://[public-ip]` in browser
    
> Sample:

<img width="1316" height="853" alt="Screenshot 2026-04-19 at 7 24 45 AM" src="https://github.com/user-attachments/assets/964c6af0-5ddc-49d8-bf73-bb6c9c865f4b" />


  - Navigate to **Target Groups > web-target-group > Targets tab** and confirm both instances show **Healthy** status

> *Screenshot: Target group showing healthy instances [Mandatory]*

> Sample:

<img width="1459" height="593" alt="Screenshot 2026-04-19 at 7 26 15 AM" src="https://github.com/user-attachments/assets/92986096-ddd7-4e0e-8f11-ce09f14d3660" />


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

<img width="1314" height="856" alt="Screenshot 2026-04-19 at 7 28 26 AM" src="https://github.com/user-attachments/assets/03dff8db-4b4b-414f-a323-cfc724706153" />


<img width="1304" height="857" alt="Screenshot 2026-04-19 at 7 27 23 AM" src="https://github.com/user-attachments/assets/723c5976-80cd-4bd0-b205-51f415e8ea6b" />


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
    
> Sample [Mandatory]:

<img width="1459" height="346" alt="Screenshot 2026-04-17 at 11 53 26 AM" src="https://github.com/user-attachments/assets/86ea0cc3-8a42-47ad-8d5f-787e1e5e1caa" />

<img width="1457" height="489" alt="Screenshot 2026-04-19 at 7 41 32 AM" src="https://github.com/user-attachments/assets/741afd61-8966-4d3c-9427-f2abcbc3d74b" />


  - Within 3-5 minutes, observe the new instance launching
    
> Sample [Mandatory]:

<img width="1459" height="294" alt="Screenshot 2026-04-19 at 7 42 00 AM" src="https://github.com/user-attachments/assets/36020e21-0ee7-4a04-8eff-02d3b20fdbe4" />


  - Monitor **CloudWatch > Alarms** to see CPU alarm trigger
    
> Sample [Mandatory]:

<img width="1459" height="402" alt="Screenshot 2026-04-19 at 7 47 02 AM" src="https://github.com/user-attachments/assets/ac670a86-99fe-48bb-a20a-e9ecbbca91d2" />


**Step 9: Observe Scale In Behavior**
  - Allow stress test to complete (5 minutes)
  - Return to **Auto Scaling Groups > web-asg > Activity tab**
  - After the cooldown period (5-10 minutes), observe the instance termination
  - Verify desired capacity returns to 2

>Sample [Mandatory]:

<img width="1444" height="528" alt="Screenshot 2026-04-19 at 8 43 56 AM" src="https://github.com/user-attachments/assets/bad14519-5e0d-4381-bc78-c2a790bc24be" />


**Step 10: Test Self Healing**
  - Navigate to **EC2 > Instances**
  - Select one Auto Scaling managed instance
  - **Instance state > Terminate instance > Terminate**
  - Check **Auto Scaling Groups > web-asg > Activity tab**
  - Confirm new instance launches automatically within 2-3 minutes

> Sample [Mandatory]:

![Screenshot 2026-04-19 at 8 45 06 AM](https://github.com/user-attachments/assets/ab496937-f3be-4188-b595-bd272081f852)

---

**Results**

The Application Load Balancer successfully distributed web traffic across multiple EC2 instances. The Auto Scaling group maintained the configured desired capacity and automatically scaled out when the simulated CPU load exceeded the 50 percent target threshold. After the load subsided, the group scaled back in, demonstrating cost efficiency. The system also demonstrated self healing behavior by automatically replacing a manually terminated instance.

---

**Discussion and Conclusion**

This lab illustrated two fundamental capabilities of cloud infrastructure that are enabled by virtualization. The Application Load Balancer provided a single point of entry while distributing requests across a pool of instances, eliminating single points of failure and improving responsiveness. Auto Scaling delivered horizontal elasticity by dynamically matching the number of running instances to real time demand. The self healing nature of the Auto Scaling group further enhanced system resilience. The combination of load balancing and auto scaling forms the foundation for designing highly available, scalable, and cost effective applications in the cloud.

---

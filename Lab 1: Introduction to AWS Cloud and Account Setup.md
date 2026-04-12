# Title: Getting Started with AWS Cloud Platform

**Objectives:**
  - Understand the AWS Cloud ecosystem and its global infrastructure
  - Create and configure an AWS Free Tier account
  - Explore the AWS Management Console and core service categories
  - Relate AWS services to cloud computing characteristics (on-demand, scalability, pay as you go)

**Theory:**

Cloud computing is the delivery of computing resources (servers, storage, databases, networking, and software) over the internet. AWS (Amazon Web Services) is the leading cloud platform, offering 200+ services from data centers around the world. The evolution of cloud computing progressed from mainframes to client-server architecture, then to virtualization, and finally to cloud computing. Key characteristics defined by NIST include:

1. **On-demand self service** – Users provision resources without human intervention
2. **Broad network access** – Services accessible over the internet
3. **Resource pooling** – Multi-tenant model serving multiple consumers
4. **Rapid elasticity** – Resources scale up or down automatically
5. **Measured service** – Pay-per-use billing model

AWS organizes its infrastructure into **Regions** (geographic areas) and **Availability Zones** (isolated data centers within a region), providing high availability and fault tolerance. The AWS Free Tier allows new users to explore services at no cost for 12 months.

**Procedure:**

**Step 1:** Open a web browser and navigate to [https://aws.amazon.com/free](https://aws.amazon.com/free)

**Step 2:** Click **"Create a Free Account"** and provide:
  - Email address
  - AWS account name
  - A strong root password

> *Screenshot: Account creation page [Optional]*

**Step 3:** Provide contact information (select "Personal" as the account type) and enter a payment method (credit or debit card for identity verification; no charges apply under the Free Tier).

**Step 4:** Complete identity verification via phone or SMS.

**Step 5:** Select the **"Basic Support - Free"** plan and click **Complete Sign Up**.

> *Screenshot: Support plan selection page [Optional]*

**Step 6:** Sign in to the **AWS Management Console** at [https://console.aws.amazon.com](https://console.aws.amazon.com)

**Step 7:** Explore the Console Dashboard:
- Locate the **Region selector** in the top right corner and select the nearest region
- Browse service categories: Compute, Storage, Database, Networking, Security

> *Screenshot: AWS Management Console dashboard showing service categories [Optional]*

**Step 8:** Navigate to **AWS Billing Dashboard** and open **Free Tier Usage** to view available free services and their usage limits.

> *Screenshot: Free Tier usage tracking page [Optional]*

**Step 9:** Open **IAM (Identity and Access Management)** and create an IAM user with admin permissions. Using an IAM user instead of the root account is a standard security practice.
- Go to IAM > Users > Add User
- Username: `lab-admin`
- Enable "AWS Management Console access"
- Attach policy: `AdministratorAccess`

> *Screenshot: IAM user creation with policy attached [Optional]*

**Step 10:** Enable **MFA (Multi-Factor Authentication)** on the root account:
- IAM > Dashboard > "Activate MFA on your root account"

> *Screenshot: MFA setup completion [Optional]*

**Results:**
  - AWS Free Tier account successfully created and configured
  - AWS Management Console explored with an understanding of service categories
  - IAM user created with appropriate permissions
  - MFA enabled on the root account
  - AWS Regions and Availability Zones structure identified

> *Screenshot: Screenshot of your AWS Management Console displaying your name and college name. [Mandatory]*

> Sample:

<img width="1464" height="846" alt="Screenshot 2026-04-12 at 9 55 33 AM" src="https://github.com/user-attachments/assets/6981ce78-5293-4248-8245-e0bbd700ac04" />



**Discussion and Conclusion:**

This lab covered the foundational setup of the AWS cloud platform, which reflects all five NIST characteristics of cloud computing. The Management Console provides on-demand self service provisioning. The global infrastructure of Regions and Availability Zones demonstrates resource pooling and broad network access. The Free Tier model reflects a measured service approach. Rapid elasticity will be observed in subsequent labs. Creating a dedicated IAM user and enabling MFA on the root account follows the principle of least privilege, which is a core security practice in any cloud environment. AWS provides a practical and accessible platform for building hands-on experience with cloud computing concepts.

---

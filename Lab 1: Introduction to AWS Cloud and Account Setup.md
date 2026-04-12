# Title: Getting Started with AWS Cloud Platform

**Objectives:**
  - Understand the AWS Cloud ecosystem and its global infrastructure
  - Create and configure an AWS Free Tier account
  - Explore the AWS Management Console and core service categories
  - Relate AWS services to cloud computing characteristics (on demand, scalability, pay as you go)

**Theory:**

Cloud computing is the delivery of computing resources servers, storage, databases, networking, software over the internet ("the cloud"). AWS (Amazon Web Services) is the leading cloud platform, offering 200+ services from data centers globally. The evolution of cloud computing moved from mainframes to client-server to virtualization to cloud. Key characteristics defined by NIST include:

  1. **On demand self service**: Users provision resources without human intervention
  2. **Broad network access**: Services accessible over the internet
  3. **Resource pooling**: Multi tenant model serving multiple consumers
  4. **Rapid elasticity**: Resources scale up/down automatically
  5. **Measured service**: Pay per use billing model

AWS organizes its infrastructure into **Regions** (geographic areas) and **Availability Zones** (isolated data centers within a region), providing high availability and fault tolerance. The AWS Free Tier allows new users to explore services at no cost for 12 months.

**Procedure:**

**Step 1:** Open a web browser and navigate to [https://aws.amazon.com/free](https://aws.amazon.com/free)

**Step 2:** Click **"Create a Free Account"** and provide:
  - Email address
  - AWS account name
  - Set a strong root password

>*Take a screenshot of the account creation page.*

**Step 3:** Provide contact information (select "Personal" account type), then enter a payment method (credit/debit card for identity verification: no charges apply under the Free Tier).

**Step 4:** Complete identity verification via phone or SMS.

**Step 5:** Select the **"Basic Support: Free"** plan and click **Complete Sign Up**.

*Take a screenshot of the support plan selection page.*

**Step 6:** Sign in to the **AWS Management Console** at [https://console.aws.amazon.com](https://console.aws.amazon.com)

**Step 7:** Explore the Console Dashboard:
  - Note the **Region selector** (top right corner) and select the region nearest to your location
  - Browse the service categories: Compute, Storage, Database, Networking, and Security

>*Take a screenshot of the AWS Management Console dashboard showing the service categories.*

**Step 8:** Navigate to **AWS Billing Dashboard** and open the **Free Tier Usage** section to review the available free services and their usage limits.

>*Take a screenshot of the Free Tier usage tracking page.*

**Step 9:** Open **IAM (Identity and Access Management)** and create an IAM user with administrator permissions rather than continuing to use the root account, which is the recommended security practice.
  - Go to IAM -> Users -> Add User
  - Username: `lab-admin`
  - Enable "AWS Management Console access"
  - Attach policy: `AdministratorAccess`

>*Take a screenshot of the IAM user creation screen with the policy attached.*

**Step 10:** Enable **MFA (Multi Factor Authentication)** on the root account by navigating to IAM -> Dashboard -> "Activate MFA on your root account" and completing the setup.

>*Take a screenshot confirming that MFA setup is complete.*

**Results:**
- AWS Free Tier account successfully created and configured
- AWS Management Console explored with an understanding of service categories
- IAM user created with appropriate permissions
- MFA enabled for root account security
- Identified AWS Regions and Availability Zones structure

**Discussion and Conclusion:**

This lab introduced the AWS cloud platform, which exemplifies all five NIST characteristics of cloud computing. The Management Console provides on demand self service provisioning. The global infrastructure of Regions and Availability Zones demonstrates resource pooling and broad network access. The Free Tier model reflects measured service, and throughout subsequent labs, rapid elasticity will be observed in practice. Creating an IAM user and enabling MFA follows the principle of least privilege, which is a critical security practice in cloud environments. AWS serves as an ideal platform for hands-on exploration of cloud computing concepts.

---

# Day 1: Introduction to AWS and Public Cloud

## 1. Understanding Cloud Computing
Historically, organizations needed to buy physical servers (often from companies like IBM or HP) to deploy their applications. These servers were stored in a data center, which required complex infrastructure management, including wiring, networking, temperature control, and configuration.

### The Problem with Traditional Servers
A major issue with buying physical servers was resource waste. If a server had 100GB of RAM and 100 CPU but only ran one application using 1GB RAM and 1 CPU, the remaining resources were wasted, making the setup very costly.

### The Solution: Virtualization
Virtualization was introduced to solve the problem of wasting resources. This concept allows for the creation of virtual servers (or virtual layers) on top of a single physical server. Instead of buying 15 physical servers for 15 applications, an organization could buy one highly configured server and deploy all 15 applications on separate virtual servers.

### Defining "The Cloud"
The concept of "Cloud" emerged because users, like developers, might request virtual machines but they do not know where exactly those resources are physically located. These resources are interconnected, hence the term "Cloud".


----
## 2. Private Cloud vs. Public Cloud

### Private Cloud
A Private Cloud setup is where an organization manages and maintains the entire cloud platform and its associated data centers internally. This setup is considered private because resources are contained within the boundaries of the organization, and no one from outside organizations can request or use those instances. Companies requiring high security, such as banking sectors, often choose to maintain private clouds using platforms like OpenStack or VMware.
### Public Cloud
Companies like Amazon (AWS), Microsoft (Azure), and Google (GCP) saw an opportunity in the overhead required to manage private data centers.

The Public Cloud concept involves cloud providers buying servers, creating infrastructure across multiple global locations, and managing the entire data center ecosystem. Anyone in the world who creates an account with AWS, Azure, or GCP can request resources, such as an EC2 instance (which is essentially a virtual machine or server). This is called Public Cloud because it is accessible to anyone, irrespective of the organization.

* Key Difference: In a private cloud, the organization manages and maintains the platform; in a public cloud, the cloud providers manage the data centers and the ecosystem.
  
While AWS manages the underlying infrastructure, the user still maintains complete control over the resources they request. For security concerns, users can create Virtual Private Clouds (VPCs) inside the public cloud.

---

## 3. Popularity of Public Cloud and AWS

### Why Public Cloud is Popular?

The primary reason for moving to the Public Cloud is to get rid of the entire maintenance overhead. Startups and mid-scale companies find managing data centers, including staffing dedicated teams (10 to 15 people), handling 24/7 operations, power loss prevention, and security patching, to be a significant burden.

* **Simplicity:** It is very simple to onboard onto a public cloud platform; organizations just pay the money and use the resources.
* **Pricing Model:** Public cloud platforms operate on a "pay as you go" model.
* **Cost:** Although cost is a factor, getting rid of maintenance overhead is the main concern driving migration.
  
#### Why AWS is the Most Popular
AWS is popular for two main reasons:
* **First-Mover Advantage:** AWS was the first company in the market and is considered a pioneer of cloud computing, successfully starting the entire concept. Many companies began their cloud journey with AWS 10 or 12 years ago and continue using it.
* **Market Share:** Due to being the first mover, AWS holds the largest market share, followed by Azure and GCP. This large market share translates directly to more job openings for candidates skilled in AWS.
  
    AWS started with a small number of resources (called Services in AWS terms, e.g., virtual machines and storage) and now offers more than 200 services. AWS constantly expands its offerings by creating services out of popular market products, such as providing a simplified Kubernetes cluster setup.

-------
## 4. Cloud Repatriation
Cloud Repatriation is the term used when people move back from the public cloud (like AWS) to on-premises solutions or private clouds.

* **Status:** This movement is true, but the percentage of users moving back is very small, roughly one percent to two percent.

* **Reasons for Moving Back:** Users move back if they do not achieve the expected security, cost optimization, or real advantage from the public cloud platform.

* **Future Trend:** The number of people onboarding public cloud platforms is expected to continue growing because startups and mid-scale companies cannot afford the financial and maintenance requirements of building their own data centers.

----
1. Creating an AWS Account (Setup Steps)
Creating an AWS account is necessary to follow along with the series.
1. Start Sign-Up: Search for "AWS sign in" and click on "create new AWS account".
2. Root User Email: Provide a root user email address and an AWS account name (which can be changed later).
3. Verification: Click "verify email address" and input the verification code sent to the email.
4. Password: Create a strong password for the account.
5. Account Type: Select "personal" for learning or personal projects.
6. Contact Information: Provide full name, phone number, select your region (e.g., India), and provide an address (which is for billing purposes and does not have to be 100% accurate).
7. Billing Details (Crucial Step): You must provide a credit card or debit card, although not all cards are accepted.
    ◦ Purpose: This step is solely for validation to ensure the user is legit and not spamming AWS by creating hundreds of accounts.
    ◦ Validation Charge: AWS will typically deduct a very small amount (e.g., one or two rupees in India, or one US dollar) to verify the transaction; this money is returned to the user later.
    ◦ Usage Charges: If you use free resources within the provided AWS limit, you will not be charged. Even if charges occur, AWS will not automatically debit the balance from the provided card; they must be paid separately, or the account will be suspended.
8. Final Steps: After providing payment details, inputting PAN card details for tax settings (if applicable), and completing the transaction verification, the remaining steps (four out of five and five out of five) are straightforward, and the account will be ready.
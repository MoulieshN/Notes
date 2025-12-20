# AWS CLI
## What is AWS CLI?
The AWS Command Line Interface (CLI) is a unified tool used to manage AWS services programmatically. It is a python utility developed by AWS engineers that acts as an abstraction layer between the user and the AWS API.

The Problem with the UI (Console)
* Manual Effort: Creating 10 VPCs or 20 S3 buckets via the UI is slow and repetitive.
* Time-Consuming: DevOps engineers frequently receive high volumes of resource requests; doing these manually can waste entire workdays.
The Programmatic Solution: APIs
To solve the limitations of the UI, tools like AWS CLI use APIs (Application Programming Interfaces).
* An API allows you to reach applications programmatically using code (like Python or Shell scripts) instead of manual clicks.
* The CLI makes this easier by translating simple shell commands into complex HTTP API requests that AWS understands.

--------------------------------------------------------------------------------
## CLI vs. Other Automation Tools
While the CLI is powerful, it is part of a larger ecosystem of infrastructure management tools:
* **AWS CLI:** Best for quick, day-to-day actions (e.g., listing resources or checking a status) where a fast response is needed.
* **Terraform & CloudFormation (IaC):** Better for creating large-scale infrastructure stacks (e.g., VPC + Subnets + EC2 + Load Balancer) because they use structured declarative templates like YAML.
* **CDK (Cloud Development Kit):** Used for defining infrastructure using familiar programming languages.

--------------------------------------------------------------------------------
## Installation and Setup
Prerequisites
* Python: Since the AWS CLI is a python utility, it is a requirement to have Python installed on your machine.
Installation Steps
1. **Source:** Only download from official AWS Documentation to ensure you get the correct, safe version.
link: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-version.html
2. **Version:** Always install AWS CLI version 2, as version 1 is outdated.
3. **OS Recommendations:**
    ◦ Mac/Linux: Use the command line or GUI installer.
    ◦ Windows: Windows is not inherently "DevOps-friendly," so it is recommended to use Git Bash or a virtual machine (via Oracle VirtualBox) to practice in a Linux-like environment.
4. **Verification:** Run aws --version in your terminal to confirm successful installation.

--------------------------------------------------------------------------------
## Authentication: aws configure
To allow the CLI to communicate with your specific AWS account, you must authenticate using the command aws configure.
Obtaining Credentials
You cannot use your standard username and password for the CLI; instead, you need Access Keys.
* Access Key ID: Functions like a username.
* Secret Access Key: Functions like a password.
* Location: In the AWS Console, go to Security Credentials under your username.
* Security Note: Access keys are highly sensitive. If leaked, someone can take control of your account. Root user access keys are discouraged; use IAM user keys instead.
Configuration Inputs
When you run aws configure, you will be asked for:
1. Access Key ID.
2. Secret Access Key.
3. Default Region: The region where your resources should be created by default (e.g., us-east-1).
4. Output Format: JSON is the preferred format for viewing responses on the terminal.

```
chandra@nl-lt-028:~$ aws --version
aws-cli/2.22.9 Python/3.12.6 Linux/6.5.0-14-generic exe/x86_64.ubuntu.22

example2: (llist out all the s3 buckets)
chandra@nl-lt-028:~$ aws s3 ls
2025-02-21 04:49:12 agent-assist-samples
2024-11-12 12:23:37 asia-translation
2024-11-09 08:49:06 asianet-translation
2024-11-09 08:49:30 askai-audio-mum
2024-11-12 12:24:11 askai-table-images
2024-11-12 12:24:11 askai-transcripts
2024-11-09 08:49:30 askaimodels
2024-11-09 08:58:58 atnt-backup
2025-07-24 17:14:50 aws-athena-query-results-ap-south-1-353042453092
2024-12-03 16:16:31 cdn-logging-g7
2024-11-21 01:39:41 chatloadesting-browser-logs
2025-07-29 13:13:50 dev-etl-poc-cdr-sink
2025-07-23 10:01:07 dev-etl-poc-cdrs


```

--------------------------------------------------------------------------------
## Using the CLI Reference Documentation
DevOps engineers are not expected to memorize every command. The AWS CLI Command Reference is the primary resource for learning how to frame commands.
* Structure: aws <service> <action> <parameters>.
* Example (S3): To list buckets, use aws s3 ls.
* Example (EC2): To launch an instance, use aws ec2 run-instances followed by parameters like --image-id, --instance-type, and --subnet-id.
* Error Handling: If you miss a required parameter, the CLI will provide an error message (e.g., "Missing parameter: image-id") to help you correct the command.

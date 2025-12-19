# AWS S3 DEEP DIVE
## Core Concepts: Buckets and Objects
* **S3 Buckets:** When you use S3, you create a "bucket" (a container) to store your data.
* **Global Uniqueness:** Every bucket name must be globally unique across all AWS accounts in the world. Because it is globally accessible via HTTP/HTTPS, no two users can have a bucket with the same name.
* **Regional Scope:** While the service is managed globally, buckets are scoped to a specific AWS region. You should choose a region near your users to reduce latency.
* **Objects:** Anything you upload (logs, images, database dumps) is considered an "object". An object consists of the data itself and metadata (size, last modified, etc.).
    * Size Limit: A single object can be as large as 5 TB.
---
## Key Characteristics (The 5 Pillars)
* **Durability (The "11 Nines"):** AWS S3 is designed for 99.999999999% durability.
    * Meaning: If you store 1 billion objects for 100 years, you might lose only one object.
    * How: S3 automatically creates multiple copies of your objects across different Availability Zones (AZs) and data centers within a region.
* **Scalability:** S3 is virtually infinite; you can store hundreds of terabytes without needing to manage the underlying hardware.
* **Availability:** Data is designed to be accessible whenever needed, supported by AWS's massive replication infrastructure.
* **Security:** S3 offers server-side encryption by default, as well as fine-grained access control via IAM policies and Bucket Policies.
* **Cost Efficiency:** S3 operates on a "pay as you go" model and offers different Storage Classes to reduce costs based on how often you access the data.
---
## Storage Classes
AWS offers several tiers to optimize for cost versus retrieval speed:
* **S3 Standard:** Default class for frequently accessed data. Fast but most expensive per GB.
* **S3 Standard-IA (Infrequent Access):** Cheaper storage for data accessed less often but still requires rapid access.
* **S3 Glacier (Flexible/Deep Archive):** Extremely cheap but retrieval takes time (from minutes to 12-48 hours for Deep Archive). This is ideal for long-term compliance backups.
---
## Advanced Features
* **Versioning:** By default, S3 replaces files with the same name. Enabling Versioning allows S3 to keep multiple copies of the same object, similar to Git, allowing you to retrieve older versions if needed.
* **Multi-part Upload:** For large files (e.g., a 4 TB database dump), S3 can break the upload into 100 MB chunks. If the connection fails, it retries only the failed chunk rather than the whole file.
* **Static Website Hosting:** S3 can serve as a web server for static content (HTML, CSS, JS). It is a very cheap alternative to traditional hosting platforms.
# ROUTE 53 
 
Route 53, the AWS service that provides Domain Name System (DNS) as a service. The **primary function of DNS is to map or resolve domain names (like amazon.com) to their corresponding IP addresses** (the address of a load balancer or application).

## The Purpose of Route 53 (DNS as a Service)
Every application deployed on AWS, such as an EC2 instance or a Load Balancer, is assigned an IP address. However, in real-world scenarios, applications are accessed using user-friendly domain names (e.g., flipkart.com) rather than complex IP addresses. 

Route 53 addresses this need by providing the service that performs the necessary mapping.
AWS offers Route 53 because setting up, hosting, and managing DNS records can be a complicated process involving multiple external services,. Route 53 simplifies this for organizations hosting their applications on AWS.

Route 53 solves two critical problems associated with raw IP addresses:

* **1. Memorability:** IP addresses (e.g., 3.6.10.171) are difficult to remember compared to simple domain names.
* **2. Dynamics of Addresses:** IP addresses can change (be dynamic), especially if a resource is restarted or a static IP is not assigned. Domain names offer a constant, memorable reference point regardless of underlying IP changes.

## Route 53 Architecture and Traffic Flow
Within the standard VPC architecture (Public Subnet, Private Subnet, Load Balancer, Internet Gateway), Route 53 acts as the initial point of contact for external traffic,.
When a user tries to access an application using a domain name (e.g., amazon.com), Route 53 intercepts the request. It then checks its records to resolve the domain name to the IP address of the Load Balancer (or the application itself, though Load Balancer IP resolution is common in real-world scenarios),. Once resolved, the request is forwarded, and the standard VPC traffic flow proceeds.

## Key Components of Route 53
Route 53 provides three primary functions or components:

* **1. Domain Registration:** AWS allows users to purchase domain names directly through Route 53. Alternatively, domains purchased from outside sellers (like GoDaddy) can also be integrated with Route 53.
* **2. Hosted Zones:** These are required whether the domain is bought internally through AWS or externally. The hosted zone is the location where the fundamental DNS records are created and maintained,. These records define the exact mapping between the domain name and the target IP address, which Route 53 uses to resolve the request,. Hosted Zones can be public or private.
* **3. Health Checks:** Route 53 can perform health checks simultaneously on web applications or servers. These checks can be performed frequently (e.g., every one minute or five minutes) to verify if a web server is active. This information is crucial for ensuring the service forwards requests only to healthy endpoints and can be used for load balancing or failover mechanisms.
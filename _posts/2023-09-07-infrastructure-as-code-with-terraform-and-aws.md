# Infrastructure Deployment with Terraform

In the realm of cloud infrastructure orchestration, Terraform has emerged as a potent tool for achieving agility and efficiency. In this blog post, we'll delve into a project that encapsulates the power of Terraform in creating a scalable and resilient WordPress infrastructure on AWS.
Our project centers around an administrative WordPress site hosted on an EC2 instance. To ensure seamless scalability, we've harnessed the capabilities of Terraform, which orchestrates an Auto Scaling Group, expanding our WordPress fleet as demand surges.
To safeguard our WordPress configurations and secrets, AWS Elastic File System (EFS) plays a pivotal role. Terraform ensures that each new instance acquires the necessary configurations swiftly.
In this ecosystem, the cornerstone is the RDS MySQL instance, responsible for maintaining our data integrity and facilitating the core functionality of WordPress.

*I have also added a decision I have to make concerning the configuration in the Cloud*

## Software Architecture of the Project
The high-level view of the software architecture of the project consists of the following:
![](/assets/infra-as-code-with-tf-aws/software-arch.png)
- An Ubuntu virtual machine (or EC2 instance)
- A MySQL database
- A PHP + WordPress installation
- The Apache web server


## High-Level Cloud Infrastructure of the Project

![](/assets/infra-as-code-with-tf-aws/hi-level-cloud-arch.png)

First, we look at how the cloud infrastructure and part of the software architecture will pair:
- <mark style="background: #BBFABBA6;">Hosting Approach</mark>: We're going with virtual machine instances running Ubuntu for our project.
- <mark style="background: #BBFABBA6;">Redundancy for WordPress</mark>: Instead of relying on a single host, we're deploying a minimum of two virtual machine instances for WordPress. This ensures redundancy and minimizes the risk of a single point of failure.
- <mark style="background: #BBFABBA6;">Shared Storage</mark>: All WordPress code and files must be accessible across all virtual machine instances. To achieve this, we're using NFS (Network File System), a cloud provider's Platform-as-a-Service offering.
- <mark style="background: #BBFABBA6;">WordPress Installation</mark>: We'll install WordPress using Infrastructure as Code scripts from a single designated virtual machine instance, called the admin host. This helps maintain consistency and control.
- <mark style="background: #BBFABBA6;">Web Host Configuration</mark>: All other hosts (workers) should have the necessary packages and configurations ready to run WordPress. They will mount the NFS share after WordPress is successfully set up on the admin host.
- <mark style="background: #BBFABBA6;">Traffic Distribution</mark>: To ensure load balancing, we'll implement a mechanism to distribute incoming traffic evenly across our multiple virtual machine instances.
- <mark style="background: #BBFABBA6;">WordPress Administration</mark>: We need to plan how to serve traffic specifically for the WordPress administration section of our website. This could involve routing traffic to the admin host or another suitable approach.

Then, we look at how we add the database into the mix of this:
- <mark style="background: #BBFABBA6;">Database Service Choice</mark>: We're going to leverage MySQL as a service, offered by the targeted cloud services, for our project's database needs. This choice streamlines database management on our virtual machine instances.
- <mark style="background: #BBFABBA6;">Database Configuration Considerations</mark>: However, there are some important considerations to keep in mind:
	- <mark style="background: #FFB86CA6;">Endpoint and Credentials</mark>: Before initiating the initial WordPress setup, we must have the database host's endpoint and the necessary access credentials ready. This ensures a smooth bootstrap process.
	- <mark style="background: #FFB86CA6;">Access Control</mark>: To enhance security, it's essential to restrict database access to only our virtual machine instances. This means configuring the database endpoint to allow connections exclusively from our authorized servers.
	- <mark style="background: #FFB86CA6;">Database Backups</mark>: Implementing a robust database backup strategy is crucial. We should set up regular backups to safeguard our data and facilitate disaster recovery.
- By addressing these database-related points, we ensure that our project's database management is both efficient and secure.

Again, we look at how the Shared Filesystem will be represented in this:
- <mark style="background: #BBFABBA6;">Shared Storage Service Choice</mark>: We'll be utilizing an NFS (Network File System) service in an "as-a-service" model for our shared storage needs. However, some considerations are crucial:
	- <mark style="background: #FFB86CA6;">NFS Endpoint Preparation</mark>: Before initiating the WordPress installation, it's essential to know the NFS endpoints. This information is required to mount the NFS storage. WordPress must be correctly installed before we proceed to launch additional virtual machine instances, ensuring a seamless setup.
	- <mark style="background: #FFB86CA6;">Access Control for NFS</mark>: Just like with the database, it's imperative to tighten security around the NFS service. We should configure access controls to permit connections solely from trusted virtual machine instances. This safeguards our shared storage from unauthorized access, preventing any random or unauthorized browsing or downloading of filesystem contents.
- By addressing these considerations related to shared storage using NFS, we ensure a secure and efficient setup that meets our project's requirements.

A final list of things to consider for the project:
- <mark style="background: #BBFABBA6;">Private Networking</mark>: To enhance security, we require an internal network within which we can deploy our resources. This private network isolates our resources from external access, providing an added layer of protection.
- <mark style="background: #BBFABBA6;">Load Balancing</mark>: To efficiently distribute incoming traffic across our backend services, we need a Layer 7 load-balancing service. This service ensures that traffic is evenly and intelligently distributed, optimizing performance and reliability.
- <mark style="background: #BBFABBA6;">Bootstrapping</mark>: The bootstrapping process involves initializing both the software stack and WordPress itself on our virtual machine instances. This step is essential for setting up and configuring our environment effectively. It ensures that everything is in place and ready to operate as intended.

By considering these aspects, we create a well-rounded deployment strategy that prioritizes security, scalability, and reliability for our project.


## Deployment CheckList
- [x] Launch and configure private network resources in your cloud provider.
- [x] Set up the database as a service.
- [x] Implement NFS filesystem as a service.
- [x] Deploy and configure the load balancer service.
- [x] Gather information on launched services and resources.
- [x] Dynamically generate scripts to bootstrap admin and web virtual machine instances.
- [x] Launch the admin virtual machine instance with the generated script, updating the OS, installing necessary software, configuring NFS, and setting up WordPress.
- [x] Launch web/worker virtual machine instances with the generated script, updating the OS, installing required software, configuring NFS, and starting the webserver.
- [x] Register all virtual machine instances with the load balancer for traffic distribution.

## AWS Architectural Diagram
![](/assets/infra-as-code-with-tf-aws/Infra-with-terraform-and-ansible-aws-final.png)


## Cloud Configuration Decision
- Notice that I mentioned redundancy when it came to placing the RDS instances in an Availability Zone. I was confronted with a decision. My options were
	1. place each RDS instance in a different uninhabited AZ
	2. place each RDS instance in the same AZ
	3. use only one RDS instance
- Here is what I found out per each option:
	1. Having an RDS instance in a different Availability Zone ensures that the database is protected in the case of a failure. In that case, a failover can be activated and the other database in the other AZ can serve the application.
	2. Having both instances in the same Availability Zone offers minimal redundancy as both databases are prone to being lost when a failure occurs in the AZ. It does make the database highly available to the application though as the failover happens fast and there is no downtime.
	3. Having only one database does not fulfill the underlying need for redundancy.
- I made the choice to go with the first option. But with a slight twist. I am using the "eu-west-2" region and there are only 3 AZs, I had to compromise and add one RDS instance to one of the other AZs housing the EC2 instances.
- This decision saves cost, as I do not have to pay for additional AZs and also makes the RDS instances redundant. A win-win situation.


## Setting Up The Infrastructure
- Navigate into the `terrafrom` directory in the [repo](https://github.com/TaskMasterErnest/Infrastructure-As-Code-With-WordPress-And-Terraform)
- To change the default variables used in deploying the infrastructure, go to the `99-variables.tf` and `01-setup.tf` files to set up your different values.
- Make sure to have Terraform installed; initialize the Terraform project with:
```Bash
terraform init
```
- Then, to check the execution plan produced by Terraform for the project:
```Bash
terraform plan
```
- If everything is alright, go ahead and deploy the infrastructure with:
```Bash
terraform apply
```


## Accessing the WordPress Admin Instance
- There will be 3 outputs; 
	- the Address to use to access the WordPress application in the instances,
	- the username to use to Login to the WordPress admin instance,
	- the password to use to Login to the WordPress instance labelled sensitive.
- To view the password, run the command `terraform output -json`. It will output all the above outputs unto you terminal and you can use these to access the WordPress instances.
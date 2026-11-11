<!-- Paste into README.md of repo: aws-secure-cloud-architecture -->
Secure Cloud Architecture on AWS: WinLocal Giveaways Proof of Concept

Module: Security Architecture (COMP H2701), TU Dublin Type: Individual project (architecture presented as part of a group presentation) Tools: AWS VPC, EC2, Auto Scaling, Application Load Balancer, RDS MySQL, S3, CloudWatch, draw.io, Linux, SSH

Overview

WinLocal Giveaways is a fictional Dublin start-up that runs online contests. Its on-premises setup (NetScaler load balancer, Windows Server 2016 with IIS, Microsoft SQL Server) had four problems:

The load balancer was a single point of failure, so any fault meant a total outage.
The database kept running out of disk space and needed constant manual management.
Scaling was manual and capacity was fixed.
A flat network let a ransomware infection go unnoticed for ten weeks.

I designed a secure, scalable three-tier AWS replacement and built a working proof of concept (PoC) in the AWS console, within a $50 budget, to show that each problem was solved.

Architecture

Architecture diagram

The design uses one VPC (10.0.0.0/16) across two Availability Zones, split into six /24 subnets in three tiers:

Tier	Subnets	What runs there
Public	10.0.1.0/24, 10.0.2.0/24	Application Load Balancer, bastion host, NAT gateway
Private application	10.0.3.0/24, 10.0.4.0/24	Web servers in an Auto Scaling Group (no public IPs)
Private database	10.0.5.0/24, 10.0.6.0/24	RDS MySQL
What I built and tested
Component	What I did	How I tested it
Networking	Custom VPC, six subnets, Internet Gateway, NAT Gateway, public and private route tables	Checked routes and subnet associations in the VPC resource map
Bastion host	EC2 jump host with SSH allowed from one administrator IP only	Connected to the private web server only via the bastion
Web tier	Private EC2 web server, AMI of the configured server, launch template, Auto Scaling Group (min 1, max 2, 60% CPU target)	Ran the Linux stress tool; CloudWatch alarm fired and the ASG launched a new instance
Load balancing	Application Load Balancer across both public subnets, target group with health checks	Refreshed the ALB URL and saw different instance IDs serving requests
Database	RDS MySQL (db.t3.micro, 20 GiB) in a private DB subnet group	Web app displayed data from the winlocal database
Failover page	S3 static website hosting a maintenance page	Loaded the S3 endpoint in a browser
Security groups	Chained groups: internet to ALB (80/443), ALB to web (80), bastion to web (22), web to RDS (3306)	Reviewed rules in the console
Designed but not built in the PoC

These appear in the architecture diagram and report but were not deployed:

AWS WAF on the ALB, to filter SQL injection, XSS and bad bots and provide request logs (addresses the ransomware issue).
Route 53 failover routing from the ALB to the S3 maintenance page, using health checks.
RDS Multi-AZ standby. The PoC used Single-AZ to stay in budget, so the database is a single point of failure until Multi-AZ is enabled.
My contribution

This was an individual project. I produced the architecture diagram, built and tested the whole PoC, wrote the report (including justification for the extra security services, a sustainability review against the AWS Well-Architected Framework and a mapping to NIST CSF 2.0), and presented the architecture as part of a group presentation.

Key security decisions
Network segmentation: web and database tiers sit in private subnets, which directly addresses the flat-network weakness behind the ransomware incident.
Least privilege: security groups reference other security groups instead of IP ranges, so rules stay correct when the Auto Scaling Group replaces instances. A compromised web server can only reach the database on port 3306.
Single entry point for administration: servers have no public IPs, and SSH goes through one locked-down bastion host.
Problems I solved along the way
Found four leftover subnets with incorrect /20 CIDR blocks from an earlier session, deleted them and rebuilt six correct /24 subnets.
Fixed an SSH "unprotected private key file" error on the bastion by correcting the key permissions (chmod 400).
Evidence
Architecture diagram: images/architecture-diagram.png
Full report (sanitised): docs/security-architecture-report.pdf
Screenshots in images/: VPC resource map, route tables, security group rules, Auto Scaling activity log, CloudWatch alarm, ALB test, RDS-backed web page, S3 maintenance page
Outcome and what I learned

The PoC showed that each of WinLocal's four problems could be addressed: segmentation, managed database, automatic scaling and load balancing across instances. The main lessons were how layered security groups limit the damage from a compromise, and how cost constraints shape design choices (Single-AZ RDS in the PoC, Multi-AZ recommended for production).

Note: Account IDs, IP addresses, resource IDs and key names have been removed or blurred from all published files. This is academic work completed in an AWS Academy lab environment.

# AWS WordPress Evolution

A comprehensive demonstration of evolving a WordPress application from a single server setup to a fully scalable, elastic architecture on AWS.

## Project Overview

This repository documents the step-by-step process of building and evolving a WordPress application on AWS, progressing from a simple, monolithic deployment to a highly available, scalable architecture. The project is structured as a hands-on learning resource for AWS Solutions Architect Associate (SAA-03) exam preparation.

## Architecture Diagram

```
+---------------------------------------------------------------------------------------------------------------------+
|                                                    AWS Cloud                                                         |
|  +---------------------------------------------------------------------------------------------------------------+  |
|  |                                         VPC (10.16.0.0/16) - A4LVPC                                           |  |
|  |                                                      |                                                         |  |
|  |                                             Internet Gateway                                                   |  |
|  |                                                      |                                                         |  |
|  |                                           Public Route Table                                                   |  |
|  |            +-----------------------+    +-----------------------+    +-----------------------+                 |  |
|  |            | Availability Zone A   |    | Availability Zone B   |    | Availability Zone C   |                 |  |
|  |            |                       |    |                       |    |                       |                 |  |
|  |            | +-------------------+ |    | +-------------------+ |    | +-------------------+ |                 |  |
|  |            | | Public Subnet A   | |    | | Public Subnet B   | |    | | Public Subnet C   | |                 |  |
|  |            | | 10.16.48.0/20     | |    | | 10.16.112.0/20    | |    | | 10.16.176.0/20    | |                 |  |
|  |            | | IPv6: [VPC]:03::/64| |    | | IPv6: [VPC]:07::/64| |    | | IPv6: [VPC]:0B::/64| |                 |  |
|  |            | +-------------------+ |    | +-------------------+ |    | +-------------------+ |                 |  |
|  |            |                       |    |                       |    |                       |                 |  |
|  |            | +-------------------+ |    | +-------------------+ |    | +-------------------+ |                 |  |
|  |            | | App Subnet A      | |    | | App Subnet B      | |    | | App Subnet C      | |                 |  |
|  |            | | 10.16.32.0/20     | |    | | 10.16.96.0/20     | |    | | 10.16.160.0/20    | |                 |  |
|  |            | | IPv6: [VPC]:02::/64| |    | | IPv6: [VPC]:06::/64| |    | | IPv6: [VPC]:0A::/64| |                 |  |
|  |            | +-------------------+ |    | +-------------------+ |    | +-------------------+ |                 |  |
|  |            |                       |    |                       |    |                       |                 |  |
|  |            | +-------------------+ |    | +-------------------+ |    | +-------------------+ |                 |  |
|  |            | | DB Subnet A       | |    | | DB Subnet B       | |    | | DB Subnet C       | |                 |  |
|  |            | | 10.16.16.0/20     | |    | | 10.16.80.0/20     | |    | | 10.16.144.0/20    | |                 |  |
|  |            | | IPv6: [VPC]:01::/64| |    | | IPv6: [VPC]:05::/64| |    | | IPv6: [VPC]:09::/64| |                 |  |
|  |            | +-------------------+ |    | +-------------------+ |    | +-------------------+ |                 |  |
|  |            +-----------------------+    +-----------------------+    +-----------------------+                 |  |
|  |                                                                                                               |  |
|  |  +--------------------------------------------------------------------------------------------------+        |  |
|  |  |                                     Security Groups                                               |        |  |
|  |  |                                                                                                  |        |  |
|  |  | +-----------------+  +----------------+  +---------------------+  +-----------------+            |        |  |
|  |  | | SGWordpress     |  | SGDatabase     |  | SGLoadBalancer      |  | SGEFS           |            |        |  |
|  |  | | Allow HTTP (80) |  | Allow MySQL    |  | Allow HTTP (80)     |  | Allow NFS (2049)|            |        |  |
|  |  | | from anywhere   |  | (3306) from WP |  | from anywhere       |  | from WP        |            |        |  |
|  |  | +-----------------+  +----------------+  +---------------------+  +-----------------+            |        |  |
|  |  +--------------------------------------------------------------------------------------------------+        |  |
|  +---------------------------------------------------------------------------------------------------------------+  |
+---------------------------------------------------------------------------------------------------------------------+
```

*Note: The diagram shows the foundational VPC architecture. As you progress through the stages, you'll add EC2 instances, RDS, EFS, Load Balancers, and Auto Scaling Groups to this infrastructure.*

## Architecture Evolution

The demo consists of 6 stages, each implementing additional components of the architecture:

1. **Stage 1** - Setup the environment and manually build WordPress on a single EC2 instance
2. **Stage 2** - Automate the build using a Launch Template
3. **Stage 3** - Split out the database into RDS and update the Launch Template
4. **Stage 4** - Split out the WordPress filesystem into EFS and update the Launch Template
5. **Stage 5** - Enable elasticity via an Auto Scaling Group & Application Load Balancer and fix WordPress (hardcoded WPHOME)
6. **Stage 6** - Cleanup

## Repository Structure

```
aws-wordpress-evolution/
├── stage1-single-server/
│   ├── aws-wordpress-vpc-architecture.md   # VPC architecture diagram
│   ├── wordpress-evolution-instructions.md # Step-by-step setup guide
│   ├── cloudformation-resources.md         # Resources created by CloudFormation
│   └── screenshots/                        # Visual documentation
├── stage2-automation/
│   └── [future content]
├── stage3-rds-integration/
│   └── [future content]
├── stage4-efs-implementation/
│   └── [future content]
├── stage5-elasticity/
│   └── [future content]
└── README.md                               # This file
```

## Key Learning Points

By working through this project, you'll learn about:

- VPC design with public and private subnets across multiple AZs
- Security groups and network access control
- EC2 instance setup and configuration
- Using Systems Manager Parameter Store for configuration
- Database setup and migration to RDS
- Shared storage with EFS
- Load balancing with ALB
- Auto Scaling for high availability and elasticity
- WordPress configuration in a distributed environment

## Target Audience

This project is designed for:
- AWS certification candidates (especially SAA-03)
- Cloud engineers looking to understand AWS architecture evolution
- DevOps professionals learning infrastructure as code
- Anyone interested in running WordPress in a scalable, production-ready environment

## Getting Started

Start with the [Stage 1 instructions](./stage1-single-server/wordpress-evolution-instructions.md) to build the initial WordPress environment. Each subsequent stage will build upon the previous one, introducing new components and concepts.

## Prerequisites

- AWS account with administrative access
- Basic understanding of AWS services
- Familiarity with Linux command line
- Understanding of WordPress fundamentals

## Important: Resource Cleanup

⚠️ **REMEMBER**: When you've completed this lab, make sure to clean up all AWS resources to avoid unnecessary charges:

1. Delete any CloudFormation stacks you've created
2. Terminate any running EC2 instances
3. Delete EFS filesystems (check for mount targets first)
4. Delete RDS instances/clusters
5. Delete any remaining EBS volumes
6. Delete load balancers and target groups
7. Check for any Route53, CloudFront, or other resources created

Stage 6 includes detailed cleanup instructions, but it's important to be thorough to avoid unexpected AWS billing charges. AWS resources continue to incur costs as long as they exist in your account, even if you're not actively using them.

## Disclaimer

This is a learning resource. While the architecture evolves toward production readiness, additional security hardening and optimizations would be recommended for actual production deployments.
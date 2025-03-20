# WordPress VPC Architecture Diagram

This diagram illustrates the VPC architecture for the WordPress Evolution project designed for AWS SAA-03 exam preparation.

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

## Key Components

1. **VPC**: A dedicated virtual network with CIDR block 10.16.0.0/16
2. **Internet Gateway**: Provides internet connectivity to the VPC
3. **Three Availability Zones**: For high availability and fault tolerance
4. **Three-Tier Architecture**:
   - Public subnets (for load balancers)
   - Application subnets (for WordPress)
   - Database subnets (for database instances)
5. **Security Groups**:
   - WordPress security group (HTTP traffic)
   - Database security group (MySQL traffic)
   - Load Balancer security group (HTTP traffic)
   - EFS security group (NFS traffic)

## Exam Tips

- This architecture demonstrates **high availability** across multiple AZs
- IPv6 is configured throughout the entire infrastructure
- Security groups are designed with **least privilege** access
- The three-tier design allows for proper **isolation of resources**
- Public subnets have direct internet access, while app and DB tiers are more protected
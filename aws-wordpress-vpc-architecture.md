# WordPress VPC Architecture Diagram

This diagram illustrates the VPC architecture for the WordPress Evolution project designed for AWS SAA-03 exam preparation.

```mermaid
graph TB
    %% Define AWS Cloud
    subgraph AWS["AWS Cloud"]
        %% Define VPC
        subgraph VPC["VPC (10.16.0.0/16) - A4LVPC"]
            IGW[Internet Gateway]
            RT[Public Route Table]
            
            %% Define Availability Zones
            subgraph AZA["Availability Zone A"]
                SNPUBA["Public Subnet A<br/>10.16.48.0/20<br/>IPv6: [VPC]:03::/64"]
                SNAPPA["App Subnet A<br/>10.16.32.0/20<br/>IPv6: [VPC]:02::/64"]
                SNDBA["DB Subnet A<br/>10.16.16.0/20<br/>IPv6: [VPC]:01::/64"]
            end
            
            subgraph AZB["Availability Zone B"]
                SNPUBB["Public Subnet B<br/>10.16.112.0/20<br/>IPv6: [VPC]:07::/64"]
                SNAPPB["App Subnet B<br/>10.16.96.0/20<br/>IPv6: [VPC]:06::/64"]
                SNDBB["DB Subnet B<br/>10.16.80.0/20<br/>IPv6: [VPC]:05::/64"]
            end
            
            subgraph AZC["Availability Zone C"]
                SNPUBC["Public Subnet C<br/>10.16.176.0/20<br/>IPv6: [VPC]:0B::/64"]
                SNAPPC["App Subnet C<br/>10.16.160.0/20<br/>IPv6: [VPC]:0A::/64"]
                SNDBC["DB Subnet C<br/>10.16.144.0/20<br/>IPv6: [VPC]:09::/64"]
            end
            
            %% Define Security Groups
            subgraph SGs["Security Groups"]
                SGWP["SGWordpress<br/>Allow HTTP (80) from anywhere"]
                SGDB["SGDatabase<br/>Allow MySQL (3306) from SGWordpress"]
                SGLB["SGLoadBalancer<br/>Allow HTTP (80) from anywhere"]
                SGEFS["SGEFS<br/>Allow NFS (2049) from SGWordpress"]
            end
        end
    end
    
    %% Define connections
    Internet((Internet)) --> IGW
    IGW --> RT
    RT --> SNPUBA
    RT --> SNPUBB
    RT --> SNPUBC
    
    classDef vpc fill:#E8F4FA,stroke:#147EB3,stroke-width:2px;
    classDef az fill:#F7F7F7,stroke:#999999,stroke-width:1px,stroke-dasharray: 5 5;
    classDef pubSubnet fill:#99C1DC,stroke:#147EB3,stroke-width:2px;
    classDef appSubnet fill:#B5DAE6,stroke:#147EB3,stroke-width:2px;
    classDef dbSubnet fill:#D1EEEE,stroke:#147EB3,stroke-width:2px;
    classDef sg fill:#FFFFFF,stroke:#232F3E,stroke-width:1px;
    classDef igw fill:#E8F4FA,stroke:#147EB3,stroke-width:2px;
    
    class VPC vpc;
    class AZA,AZB,AZC az;
    class SNPUBA,SNPUBB,SNPUBC pubSubnet;
    class SNAPPA,SNAPPB,SNAPPC appSubnet;
    class SNDBA,SNDBB,SNDBC dbSubnet;
    class SGs,SGWP,SGDB,SGLB,SGEFS sg;
    class IGW igw;
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
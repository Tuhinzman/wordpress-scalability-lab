# WordPress VPC - CloudFormation Resources List

This document details all the AWS resources that will be created by the CloudFormation template for the WordPress Architecture Evolution project. It's an excellent reference for AWS SAA-03 exam preparation to understand infrastructure as code and resource relationships.

## Network Resources

### 1. VPC
- **Resource Type**: `AWS::EC2::VPC`
- **Name**: A4LVPC
- **IPv4 CIDR**: 10.16.0.0/16
- **Features**: DNS support and DNS hostnames enabled

### 2. IPv6 CIDR Block
- **Resource Type**: `AWS::EC2::VPCCidrBlock`
- **Purpose**: Enables IPv6 for the VPC
- **Configuration**: Amazon provided IPv6 CIDR block (/56)

### 3. Internet Gateway
- **Resource Type**: `AWS::EC2::InternetGateway`
- **Name**: A4L-IGW
- Connected to the VPC via a `VPCGatewayAttachment` resource

### 4. Route Table
- **Resource Type**: `AWS::EC2::RouteTable`
- **Name**: A4L-vpc-rt-pub
- **Purpose**: Routes traffic for public subnets
- **Default Routes**:
  - IPv4 (0.0.0.0/0) → Internet Gateway
  - IPv6 (::/0) → Internet Gateway

## Subnet Resources

### Public Subnets (3)
1. **SNPUBA (sn-pub-A)**
   - **Resource Type**: `AWS::EC2::Subnet`
   - **Availability Zone**: First AZ
   - **IPv4 CIDR**: 10.16.48.0/20
   - **IPv6 CIDR**: [VPC IPv6]:03::/64
   - **Features**: Auto-assigns public IPv4 addresses

2. **SNPUBB (sn-pub-B)**
   - **Resource Type**: `AWS::EC2::Subnet`
   - **Availability Zone**: Second AZ
   - **IPv4 CIDR**: 10.16.112.0/20
   - **IPv6 CIDR**: [VPC IPv6]:07::/64
   - **Features**: Auto-assigns public IPv4 addresses

3. **SNPUBC (sn-pub-C)**
   - **Resource Type**: `AWS::EC2::Subnet`
   - **Availability Zone**: Third AZ
   - **IPv4 CIDR**: 10.16.176.0/20
   - **IPv6 CIDR**: [VPC IPv6]:0B::/64
   - **Features**: Auto-assigns public IPv4 addresses

### Application Subnets (3)
1. **SNAPPA (sn-app-A)**
   - **Resource Type**: `AWS::EC2::Subnet`
   - **Availability Zone**: First AZ
   - **IPv4 CIDR**: 10.16.32.0/20
   - **IPv6 CIDR**: [VPC IPv6]:02::/64

2. **SNAPPB (sn-app-B)**
   - **Resource Type**: `AWS::EC2::Subnet`
   - **Availability Zone**: Second AZ
   - **IPv4 CIDR**: 10.16.96.0/20
   - **IPv6 CIDR**: [VPC IPv6]:06::/64

3. **SNAPPC (sn-app-C)**
   - **Resource Type**: `AWS::EC2::Subnet`
   - **Availability Zone**: Third AZ
   - **IPv4 CIDR**: 10.16.160.0/20
   - **IPv6 CIDR**: [VPC IPv6]:0A::/64

### Database Subnets (3)
1. **SNDBA (sn-db-A)**
   - **Resource Type**: `AWS::EC2::Subnet`
   - **Availability Zone**: First AZ
   - **IPv4 CIDR**: 10.16.16.0/20
   - **IPv6 CIDR**: [VPC IPv6]:01::/64

2. **SNDBB (sn-db-B)**
   - **Resource Type**: `AWS::EC2::Subnet`
   - **Availability Zone**: Second AZ
   - **IPv4 CIDR**: 10.16.80.0/20
   - **IPv6 CIDR**: [VPC IPv6]:05::/64

3. **SNDBC (sn-db-C)**
   - **Resource Type**: `AWS::EC2::Subnet`
   - **Availability Zone**: Third AZ
   - **IPv4 CIDR**: 10.16.144.0/20
   - **IPv6 CIDR**: [VPC IPv6]:09::/64

## Security Resources

### Security Groups (4)
1. **SGWordpress**
   - **Resource Type**: `AWS::EC2::SecurityGroup`
   - **Purpose**: Controls access to WordPress instances
   - **Inbound Rules**: Allow HTTP (port 80) from anywhere (0.0.0.0/0)

2. **SGDatabase**
   - **Resource Type**: `AWS::EC2::SecurityGroup`
   - **Purpose**: Controls access to database instances
   - **Inbound Rules**: Allow MySQL (port 3306) from SGWordpress only

3. **SGLoadBalancer**
   - **Resource Type**: `AWS::EC2::SecurityGroup`
   - **Purpose**: Controls access to load balancers
   - **Inbound Rules**: Allow HTTP (port 80) from anywhere (0.0.0.0/0)

4. **SGEFS**
   - **Resource Type**: `AWS::EC2::SecurityGroup`
   - **Purpose**: Controls access to Elastic File System
   - **Inbound Rules**: Allow NFS/EFS (port 2049) from SGWordpress only

## IAM Resources

1. **WordpressRole**
   - **Resource Type**: `AWS::IAM::Role`
   - **Purpose**: Role for WordPress EC2 instances
   - **Managed Policies**: 
     - CloudWatchAgentServerPolicy
     - AmazonSSMFullAccess
     - AmazonElasticFileSystemClientFullAccess

2. **WordpressInstanceProfile**
   - **Resource Type**: `AWS::IAM::InstanceProfile`
   - **Purpose**: Instance profile that attaches the WordpressRole to EC2 instances

3. **IPv6WorkaroundRole**
   - **Resource Type**: `AWS::IAM::Role`
   - **Purpose**: Role for the Lambda function that configures IPv6
   - **Policies**: 
     - Allows CloudWatch logging
     - Allows EC2 subnet modifications

## Lambda Resources

1. **IPv6WorkaroundLambda**
   - **Resource Type**: `AWS::Lambda::Function`
   - **Purpose**: Enables IPv6 address auto-assignment on public subnets
   - **Runtime**: Python 3.9
   - **Timeout**: 30 seconds

## Custom Resources

Three custom resources that use the Lambda function to configure IPv6 on the public subnets:
1. **IPv6WorkaroundSubnetPUBA**
2. **IPv6WorkaroundSubnetPUBB**
3. **IPv6WorkaroundSubnetPUBC**

## Monitoring Resources

1. **CWAgentConfig**
   - **Resource Type**: `AWS::SSM::Parameter`
   - **Purpose**: Stores CloudWatch Agent configuration
   - **Monitoring**:
     - System logs (/var/log/secure)
     - Apache logs (access_log, error_log)
     - System metrics (CPU, memory, disk, network)

## Route Table Associations

6 subnet-route table associations connecting the public subnets to the public route table:
1. **RTAssociationPubA**
2. **RTAssociationPubB**
3. **RTAssociationPubC**

## Exam Tips

Here are some key points to remember for the AWS SAA-03 exam:

1. **Three-Tier Architecture**:
   - Public tier for load balancers
   - Application tier for WordPress
   - Database tier for databases
   - Each in their own subnets across AZs

2. **Security Group Design**:
   - Remember the principle of least privilege
   - Database only accepts connections from the application tier
   - EFS only accepts connections from the application tier

3. **IPv6 Configuration**:
   - IPv6 is enabled throughout the infrastructure
   - Custom resources with Lambda are used to configure IPv6 on subnets

4. **High Availability**:
   - Resources spread across three Availability Zones
   - This design supports building highly available applications

5. **CloudFormation**:
   - Understand dependencies between resources
   - Note how the template uses intrinsic functions like `!Ref`, `!GetAtt`, and `!Select`

6. **Networking**:
   - Understand CIDR block allocation and subnet sizing
   - Public subnets have routes to the internet via the Internet Gateway
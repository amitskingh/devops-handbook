# AWS EC2 — Notes

> Learning perspective: Cloud Engineering & DevOps  
> Focus: Fundamentals, architecture, networking, hands-on AWS, security, and interview preparation.

---

## 1. What is AWS EC2?

**Amazon EC2 (Elastic Compute Cloud)** is an AWS service that provides virtual servers called **instances**.

Instead of purchasing and maintaining physical servers, you can provision virtual machines on AWS infrastructure.

### Traditional server vs EC2

| Traditional server | AWS EC2 |
|---|---|
| Purchase physical hardware | Provision virtual instances |
| Manually maintain hardware | AWS manages underlying infrastructure |
| Limited hardware capacity | Select different instance sizes |
| Scaling takes time | Launch or terminate instances quickly |
| Large upfront investment | Pay according to selected pricing model |

### Real-world example

Suppose you want to deploy a Node.js application.

Traditional approach:

1. Purchase a physical server.
2. Install Linux.
3. Configure networking.
4. Install Node.js.
5. Deploy the application.

AWS approach:

1. Launch an EC2 instance.
2. Select an operating system.
3. Configure networking and security.
4. Install Node.js.
5. Deploy the application.

---

## 2. EC2 Architecture

```text
                    AWS Cloud
                       |
                       v
                 AWS Region
                       |
                       v
                Availability Zone
                       |
                       v
                     VPC
                       |
                       v
                    Subnet
                       |
                       v
                 EC2 Instance
                       |
                       v
                  Application
```

### Important components

| Component | Purpose |
|---|---|
| Region | Geographic AWS location |
| Availability Zone (AZ) | Isolated infrastructure location within a Region |
| VPC | Logically isolated virtual network |
| Subnet | IP address range within a VPC, located in one AZ |
| EC2 | Virtual server |
| Security Group | Virtual firewall for an EC2 instance |
| EBS | Persistent block storage |
| AMI | Template used to launch an EC2 instance |
| Key Pair | Credentials used for SSH authentication |

---

## 3. EC2 Instance Types

EC2 instances come in different families, optimized for different workloads.

| Family | Purpose | Example workload |
|---|---|---|
| General Purpose | Balanced CPU, memory, networking | Web applications |
| Compute Optimized | High CPU performance | Processing, compute-heavy tasks |
| Memory Optimized | High memory capacity | In-memory databases |
| Storage Optimized | High storage performance | Data-intensive workloads |
| Accelerated Computing | GPU and specialized hardware | AI/ML, graphics processing |

### Instance size

Example:

` t3.micro `

- `t` = instance family
- `3` = generation
- `micro` = instance size

The actual resources depend on the specific instance type.

---

## 4. AMI — Amazon Machine Image

An **AMI** is a template used to launch an EC2 instance.

It contains information such as:

- Operating system
- Preinstalled software
- Configuration
- Storage setup

Examples:

- Amazon Linux
- Ubuntu Server
- Windows Server

### AMI workflow

```text
Choose AMI
    |
    v
Select Instance Type
    |
    v
Configure Network
    |
    v
Launch EC2
```

Think of an AMI as a starting template for creating a server.

---

## 5. VPC and Subnets

A VPC is your logically isolated network inside AWS.

Example:

```text
VPC: 10.0.0.0/16

    +----------------------------+
    |          VPC               |
    |                            |
    |  Public Subnet              |
    |  10.0.1.0/24                |
    |      EC2 Web Server         |
    |                            |
    |  Private Subnet             |
    |  10.0.2.0/24                |
    |      EC2 Backend            |
    |                            |
    |  Private Subnet              |
    |  10.0.3.0/24                |
    |      Database                |
    +----------------------------+
```

### VPC

A VPC provides:

- IP address ranges
- Subnets
- Route tables
- Network connectivity configuration
- Network isolation

### Subnet

A subnet is a smaller IP network within a VPC.

Example:

```text
VPC CIDR:      10.0.0.0/16
Public subnet: 10.0.1.0/24
Private subnet:10.0.2.0/24
```

Each subnet belongs to exactly one VPC and one Availability Zone.

### Important

Different VPCs can have identical subnet CIDR ranges.

Example:

```text
VPC A:
    10.0.1.0/24

VPC B:
    10.0.1.0/24
```

This is allowed because the VPCs are isolated.

However, overlapping CIDRs can cause routing problems when connecting VPCs using VPC Peering or Transit Gateway.

---

## 6. Public Subnet vs Private Subnet

A subnet's public or private classification depends primarily on its routing configuration.

### Public subnet

A public subnet has a route to an Internet Gateway.

```text
Destination: 0.0.0.0/0
Target:      Internet Gateway
```

Typical usage:

- Public web servers
- Bastion hosts
- Public-facing load balancers

### Private subnet

A private subnet does not have a direct route to an Internet Gateway.

Typical usage:

- Backend application servers
- Databases
- Internal services

Private instances can use a NAT Gateway for outbound internet access when configured.

### Important distinction

A public subnet does not automatically make every EC2 instance publicly accessible.

The instance also needs appropriate public addressing, routing, and security rules.

---

## 7. Public IP vs Private IP

An EC2 instance commonly receives a private IPv4 address.

Example:

```text
Private IP: 10.0.1.25
Public IP:  54.x.x.x
```

### Private IP

Used for communication within private networks.

Examples of private IPv4 ranges:

| Range | CIDR |
|---|---|
| 10.x.x.x | 10.0.0.0/8 |
| 172.16.x.x – 172.31.x.x | 172.16.0.0/12 |
| 192.168.x.x | 192.168.0.0/16 |

Private addresses are not directly routable over the public internet.

### Public IP

Used for communication over the public internet.

A public IP must be globally distinguishable within the relevant internet routing context.

### Auto-assign Public IP

This setting determines whether AWS automatically assigns a public IPv4 address when launching an EC2 instance.

| Setting | Result |
|---|---|
| Enabled | Instance can receive an automatically assigned public IPv4 address |
| Disabled | Instance receives no automatically assigned public IPv4 address through this setting |

The instance still receives a private IP address.

### Important

A public IP alone does not make an EC2 instance internet-accessible.

You also need:

- Public subnet
- Internet Gateway
- Correct route table
- Security Group allowing the required traffic

---

## 8. Internet Gateway (IGW)

An Internet Gateway connects a VPC to the internet.

Example:

```text
Laptop
   |
   v
Internet
   |
   v
Internet Gateway
   |
   v
VPC
   |
   v
EC2 Instance
```

An Internet Gateway supports internet connectivity for appropriately configured resources.

### Route table example

| Destination | Target |
|---|---|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |

`10.0.0.0/16` represents the VPC's local network.

`0.0.0.0/0` represents all IPv4 destinations not matched by a more specific route.

---

## 9. Security Groups

A Security Group is a virtual firewall associated with an EC2 instance or network interface.

It controls inbound and outbound traffic.

### Example Security Group

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | My IP |
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

### Important characteristics

- Security Groups are stateful.
- They support allow rules, not explicit deny rules.
- Return traffic for an allowed connection is automatically permitted.
- Multiple Security Groups can be associated with a network interface.

### Security best practice

Avoid:

```text
SSH (22)
Source: 0.0.0.0/0
```

Prefer:

```text
SSH (22)
Source: Your current public IP/32
```

This limits SSH access to your own IP address.

---

## 10. SSH — Connecting to EC2

SSH allows secure remote access to Linux servers.

Example:

```bash
ssh -i my-key.pem ubuntu@PUBLIC_IP
```

For Amazon Linux:

```bash
ssh -i my-key.pem ec2-user@PUBLIC_IP
```

For Ubuntu:

```bash
ssh -i my-key.pem ubuntu@PUBLIC_IP
```

### Requirements

- EC2 instance running
- Public IP or another reachable connection path
- SSH Security Group rule
- Correct private key
- Correct username
- Network route permitting connectivity

### Common SSH issues

| Error | Possible reason |
|---|---|
| Connection timed out | Network route or Security Group issue |
| Permission denied | Incorrect key or username |
| Host unreachable | Routing or connectivity issue |
| Connection refused | SSH service may not be running or listening |

---

## 11. EC2 Networking — How a Request Reaches the Server

Suppose your laptop accesses an EC2 web server.

```text
Laptop
   |
   v
Home Router / Mobile Carrier
   |
   v
Internet
   |
   v
AWS Public IPv4
   |
   v
Internet Gateway
   |
   v
VPC Route Table
   |
   v
EC2 Security Group
   |
   v
EC2 Instance
   |
   v
Web Application
```

### Step-by-step

1. Laptop sends a request to the EC2 public IP.
2. The internet routes traffic toward that public IP.
3. AWS delivers traffic through the Internet Gateway.
4. The VPC routing configuration determines the network path.
5. Security Group rules evaluate the traffic.
6. EC2 receives the request if permitted.
7. The application processes the request.
8. The response returns through the established network path.

### Important

Security Groups and route tables serve different purposes.

- Route table: Where should traffic go?
- Security Group: Is this traffic allowed?

---

## 12. Elastic IP

An Elastic IP (EIP) is a static public IPv4 address allocated to your AWS account.

### Automatically assigned public IP vs Elastic IP

| Feature | Auto-assigned public IP | Elastic IP |
|---|---|---|
| Static | Generally no | Yes, while allocated |
| Stop/start behavior | Can change | Can remain associated |
| Reassociation | Not designed as a persistent address | Can be reassociated with supported resources |
| Cost | Pricing applies | Pricing applies; check current AWS rates |

Elastic IPs are useful when a server needs a stable public IPv4 address.

---

## 13. EBS — Elastic Block Store

EBS provides persistent block storage for EC2.

Think of it as a virtual hard disk.

```text
EC2 Instance
    |
    +---- EBS Volume
    |
    +---- EBS Volume
```

### EBS characteristics

- Persistent block storage
- Can survive EC2 instance termination when configured appropriately
- Supports snapshots
- Used for operating systems, application files, and databases

### EBS vs Instance Store

| EBS | Instance Store |
|---|---|
| Persistent block storage | Temporary instance-attached storage |
| Can survive instance stop/start | Data may be lost when instance stops or terminates, depending on lifecycle |
| Snapshots supported | Not backed up through EBS snapshots |

---

## 14. EC2 Lifecycle

Typical EC2 states:

```text
Pending
   |
   v
Running
   |
   v
Stopping
   |
   v
Stopped
   |
   v
Terminated
```

### Stop

- Instance shuts down.
- EBS-backed data generally persists.
- Automatically assigned public IPv4 may change after restart.
- Compute billing stops, but storage and other applicable charges may continue.

### Start

- Instance starts again.
- Private IP generally remains for the same network interface.
- Automatically assigned public IPv4 may be different.

### Reboot

- Restarts the operating system.
- Instance remains allocated.
- Public IPv4 generally remains unchanged.

### Terminate

- Permanently deletes the instance.
- EBS volumes configured for deletion on termination may be deleted.
- Other resources and charges may remain.

---

## 15. EC2 Pricing Models

| Pricing model | Use case |
|---|---|
| On-Demand | Flexible workloads without long-term commitment |
| Reserved Instances | Commitment-based pricing for suitable workloads |
| Savings Plans | Flexible commitment-based compute pricing |
| Spot Instances | Interruptible workloads at discounted prices |
| Dedicated Hosts | Dedicated physical server capacity |

### Cost considerations

EC2 costs can include:

- Instance compute usage
- EBS storage
- Public IPv4 addresses
- Data transfer
- Load Balancers
- NAT Gateways
- Snapshots

Always check current AWS pricing before deploying resources.

---

## 16. High Availability and Scalability

A production application should not depend on one EC2 instance.

Typical architecture:

```text
                 Internet
                    |
                    v
              Load Balancer
                    |
          +---------+---------+
          |                   |
          v                   v
       EC2-A                EC2-B
          |                   |
          +---------+---------+
                    |
                    v
               Database
```

### Why multiple EC2 instances?

- Better availability
- Horizontal scaling
- Fault tolerance
- Load distribution

### Auto Scaling

EC2 Auto Scaling automatically adjusts instance capacity based on configured policies.

Example:

```text
Low traffic:
    2 EC2 instances

High traffic:
    5 EC2 instances

Traffic decreases:
    Scale back to 2 instances
```

---

## 17. EC2 vs Containers

| EC2 | Container |
|---|---|
| Virtual machine | Isolated application environment |
| Runs an operating system | Shares the host OS kernel |
| More infrastructure management | Usually lighter deployment |
| Can host Docker | Can run inside EC2 |

Example:

```text
EC2 Instance
    |
    v
Docker
    |
    +---- Node.js Container
    |
    +---- Redis Container
```

EC2 and Docker are not alternatives in every scenario. Docker can run on EC2.

---

## 18. Hands-on AWS Practice

### Task 1: Launch EC2

1. Open AWS Console.
2. Navigate to EC2.
3. Choose Launch Instance.
4. Select an AMI.
5. Choose an instance type.
6. Select your custom VPC.
7. Select your public subnet.
8. Enable Auto-assign Public IP for direct public IPv4 access.
9. Configure Security Group.
10. Select or create a Key Pair.
11. Launch the instance.

### Task 2: Connect to EC2

```bash
ssh -i my-key.pem ubuntu@PUBLIC_IP
```

### Task 3: Install Nginx

```bash
sudo apt update

sudo apt install nginx -y

sudo systemctl enable nginx

sudo systemctl start nginx
```

### Task 4: Access the website

Open:

```text
http://PUBLIC_IP
```

Ensure the Security Group allows inbound TCP port 80.

### Task 5: Stop the instance

Stop the instance when you're finished to reduce compute charges.

Remember that EBS, public IPv4, and other resources may still incur charges.

---

## 19. Common EC2 Troubleshooting

| Problem | What to check |
|---|---|
| Cannot SSH | Security Group, route table, public IP, key, username |
| Website unavailable | Application status, port 80/443, Security Group |
| No internet access | Route table, IGW/NAT, DNS, firewall |
| EC2 has no public IP | Public IP assignment and subnet settings |
| EC2 is unreachable | Network ACLs, Security Groups, routing |
| Application is slow | CPU, memory, disk, network, application logs |

Useful commands:

```bash
# CPU and memory
top

# Disk usage
df -h

# Memory usage
free -h

# Network interfaces
ip addr

# Routing table
ip route

# Listening ports
sudo ss -tulpn

# Nginx status
sudo systemctl status nginx
```

---

## 20. AWS EC2 Best Practices

### Security

- Use least-privilege Security Groups.
- Avoid exposing SSH to the entire internet.
- Use IAM roles for applications accessing AWS services.
- Keep operating systems updated.
- Prefer Systems Manager Session Manager where appropriate.
- Avoid hardcoding AWS credentials.

### Reliability

- Use multiple Availability Zones for production workloads.
- Consider Auto Scaling.
- Back up important EBS volumes.
- Monitor application health.

### Cost

- Stop unused development instances.
- Delete unused resources.
- Monitor public IPv4, EBS, and NAT Gateway costs.
- Use AWS Budgets and billing alerts.

### Maintainability

- Use consistent resource naming.
- Tag AWS resources.
- Automate infrastructure using Terraform or AWS CloudFormation.
- Prefer repeatable deployment workflows.

---

## 21. Important Interview Questions

### Fundamentals

1. What is AWS EC2?
2. What is an AMI?
3. What is an EC2 instance type?
4. What is the difference between EC2 and a physical server?
5. What is the difference between stopping and terminating EC2?

### Networking

6. What is a VPC?
7. What is a subnet?
8. What is the difference between public and private subnets?
9. What is an Internet Gateway?
10. What is a NAT Gateway?
11. What is a Security Group?
12. What is the difference between Security Groups and Network ACLs?
13. What happens when an EC2 instance receives a request from the internet?
14. Why does an EC2 instance need a public IP?
15. Can two VPCs have identical private IP ranges?

### Storage

16. What is EBS?
17. What is the difference between EBS and Instance Store?
18. What happens to EBS when EC2 is terminated?

### Reliability and Scaling

19. What is EC2 Auto Scaling?
20. What is a Load Balancer?
21. How do you design a highly available EC2 architecture?
22. What is the difference between vertical and horizontal scaling?

---

## 22. Final Mental Model

```text
AWS Region
    |
    v
Availability Zone
    |
    v
VPC
    |
    v
Subnet
    |
    v
EC2 Instance
    |
    +---- Security Group
    |
    +---- Private IP
    |
    +---- Optional Public IP
    |
    +---- EBS Storage
    |
    +---- Application
```

Remember:

- **EC2** = Virtual server.
- **AMI** = Template for launching a server.
- **VPC** = Your isolated virtual network.
- **Subnet** = Smaller IP network inside a VPC.
- **Internet Gateway** = VPC internet connectivity.
- **Security Group** = Instance/network-interface firewall.
- **EBS** = Persistent block storage.
- **Elastic IP** = Static public IPv4 address.
- **Auto Scaling** = Automatically adjusts instance capacity.
- **Load Balancer** = Distributes traffic across targets.

The goal is not just to memorize AWS services. Understand how networking, compute, storage, security, reliability, and cost work together to run an application in the cloud.

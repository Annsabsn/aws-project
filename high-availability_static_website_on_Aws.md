# High Availability Static Website Architecture on AWS

This project demonstrates how to build a **highly available, scalable web architecture in AWS** using **VPC, EC2, NGINX, Application Load Balancer, Auto Scaling Group, CloudWatch, and SNS**.

The architecture ensures:

* High Availability
* Automatic scaling
* Fault tolerance
* Monitoring and alerting

---

# Architecture Overview

<img width="1350" height="906" alt="Screenshot 2026-03-04 235443" src="https://github.com/user-attachments/assets/130d41aa-7e65-47ee-a40a-085001dd089b" />


Traffic Flow:

User → Load Balancer DNS → Target Group → EC2 Instances → NGINX Website

---

# Step-by-Step Implementation

---

# Step 1: Create a Custom VPC

### Purpose

A **Virtual Private Cloud (VPC)** provides an isolated network environment in AWS.

### Steps

1. Go to **VPC Console**
2. Click **Create VPC**
3. Select **VPC Only**
4. Configure:

   * Name: `inter-pro-vpc`
   * IPv4 CIDR: `10.0.0.0/16`
5. Create the VPC

### Explanation

The VPC acts as the **private network** where all AWS resources will run securely.

---

# Step 2: Create Subnets

### Purpose

Subnets divide the VPC into **smaller networks**.

### Public Subnets

Used for:

* Load Balancer
* Bastion Host

### Private Subnets

Used for:

* Application servers

### Configuration

| Subnet           | Type    | AZ          |
| ---------------- | ------- | ----------- |
| public-subnet-1  | Public  | ap-south-1a |
| public-subnet-2  | Public  | ap-south-1b |
| private-subnet-1 | Private | ap-south-1a |
| private-subnet-2 | Private | ap-south-1b |

### Explanation

Private subnets protect servers from direct internet exposure.

---

# Step 3: Create Internet Gateway

### Purpose

Allows resources in **public subnet** to access the internet.

### Steps

1. Create **Internet Gateway**
2. Attach it to the **VPC**
3. Update **Route Table**

Route:

0.0.0.0/0 → Internet Gateway

### Explanation

Without IGW, public instances cannot communicate with the internet.

---

# Step 4: Create NAT Gateway

### Purpose

Allows **private instances to access the internet** without exposing them publicly.

### Steps

1. Go to **NAT Gateway**
2. Select **Public Subnet**
3. Allocate **Elastic IP**
4. Create NAT Gateway

Update **Private Route Table**

Route:

0.0.0.0/0 → NAT Gateway

### Explanation

Private servers can download updates but remain hidden from the internet.

---

# Step 5: Launch EC2 Instances

### Public Instance

Used for SSH access.

Configuration:

* Amazon Linux / Ubuntu
* Public subnet
* Enable Public IP

### Private Instance

Used for hosting the website.

Configuration:

* Private subnet
* No public IP

---

# Step 6: Install NGINX and Deploy Website

SSH into EC2.

Install packages:

```bash
sudo apt update
sudo apt install nginx zip -y
```

Download template

```bash
wget -O app.zip https://www.tooplate.com/download/2097_pop
```

Unzip files

```bash
unzip app.zip
```

Move website files

```bash
sudo cp -r ./2097_pop/* /var/www/html/
```

Restart NGINX

```bash
sudo systemctl restart nginx
```

Explanation:

NGINX acts as the **web server** hosting the static website.

---

# Step 7: Create Application Load Balancer

Purpose:
Distributes incoming traffic across multiple EC2 instances.

Steps:

1. Go to **EC2 → Load Balancers**
2. Create **Application Load Balancer**
3. Select

   * Internet Facing
4. Choose **Public Subnets**
5. Create security group allowing HTTP (80)

Explanation:

ALB ensures **traffic distribution and high availability**.

---

# Step 8: Create Target Group

Purpose:
Defines where the Load Balancer sends traffic.

Steps:

1. Target Type: Instance
2. Protocol: HTTP
3. Port: 80
4. Health Check Path: /

Register EC2 instance.

Explanation:

Health checks ensure traffic only goes to **healthy servers**.

---

# Step 9: Create AMI

Purpose:
AMI allows quick duplication of instances.

Steps:

1. Select private EC2 instance
2. Click **Create Image**
3. Name: `ami-int`

Explanation:

AMI is used by Auto Scaling to launch new instances.

---

# Step 10: Create Launch Template

Purpose:
Defines configuration for Auto Scaling instances.

Configuration:

* AMI
* Instance type
* Security Group
* Key pair
* Network settings

Explanation:

Launch Template standardizes instance creation.

---

# Step 11: Create Auto Scaling Group

Purpose:
Automatically adds or removes EC2 instances based on demand.

Configuration:

| Setting | Value |
| ------- | ----- |
| Min     | 1     |
| Desired | 2     |
| Max     | 3     |

Attach:

* Load Balancer
* Target Group

Explanation:

Ensures **automatic scaling and fault tolerance**.

---

# Step 12: Configure CloudWatch Alarms

Purpose:
Monitor CPU usage.

Example rule:

Scale Out → CPU > 70%
Scale In → CPU < 30%

Explanation:

Triggers scaling policies automatically.

---

# Step 13: Configure SNS Notifications

Purpose:
Send alerts via email.

Steps:

1. Create SNS Topic
2. Add Email Subscription
3. Attach to Auto Scaling Group

Explanation:

Admins receive notifications when scaling events occur.

---

# Final Result

The architecture provides:

* High availability
* Auto scaling
* Fault tolerance
* Load balancing
* Monitoring and alerting

Users access the website using:

Load Balancer DNS

---

# Interview Questions

### Basic Questions

**1. What is VPC?**
A Virtual Private Cloud is an isolated network environment in AWS.

**2. What is the difference between public and private subnet?**

Public subnet:
Has route to Internet Gateway

Private subnet:
No direct internet access

---

**3. What is NAT Gateway?**

Allows private instances to access the internet securely.

---

**4. What is a Target Group?**

A group of EC2 instances that receive traffic from the Load Balancer.

---

**5. Why use Auto Scaling?**

To automatically adjust resources based on traffic demand.

---

# Advanced Interview Questions

**1. Why host application servers in private subnet?**

Security best practice.
Servers cannot be accessed directly from the internet.

---

**2. What happens if one Availability Zone fails?**

Load Balancer routes traffic to instances in the other AZ.

---

**3. Why use Launch Templates instead of Launch Configurations?**

Launch Templates support:

* Versioning
* Spot instances
* Advanced configuration

---

**4. How does ALB health check work?**

ALB sends HTTP requests to the instance health check path.

Example:

```
http://instance-ip:80/
```

If response code = 200 → healthy.

---

# Scenario-Based Questions

### Scenario 1

Traffic suddenly increases.

Solution:

Auto Scaling launches additional EC2 instances.

---

### Scenario 2

One EC2 instance fails.

Solution:

ALB stops sending traffic and ASG launches replacement instance.

---

### Scenario 3

Website becomes slow.

Possible reasons:

* CPU high
* insufficient instances
* network bottleneck

Solution:

Increase scaling policy or instance size.


Created VPC:
<img width="1909" height="761" alt="Screenshot 2026-03-04 173647" src="https://github.com/user-attachments/assets/56a71503-3a22-404f-b016-057b21d624d1" />
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
Created ec2 instances :
<img width="1906" height="870" alt="Screenshot 2026-03-04 173512" src="https://github.com/user-attachments/assets/1ea67b1a-aa5d-42ad-aec2-adad19679572" />
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Hosting website:
<img width="1885" height="642" alt="Screenshot 2026-03-04 173549" src="https://github.com/user-attachments/assets/8c04c7ee-5d77-4abe-bc45-91b0dec51ea4" />
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Load balancer:
<img width="1902" height="812" alt="Screenshot 2026-03-04 174345" src="https://github.com/user-attachments/assets/8bfba61c-9f07-4abd-9b08-b141a3188538" />
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
AMI:
<img width="1898" height="872" alt="Screenshot 2026-03-04 174328" src="https://github.com/user-attachments/assets/ea4123f0-22a4-41cc-99f2-81783673f518" />
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
Launch Template:
<img width="1911" height="811" alt="Screenshot 2026-03-04 174534" src="https://github.com/user-attachments/assets/505608f2-4ea4-4272-aeb9-bca4f07d3bc7" />
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Auto Scaling Group:
<img width="1914" height="822" alt="Screenshot 2026-03-04 175659" src="https://github.com/user-attachments/assets/995ac0ee-4902-48c7-9c7d-f99715cfa932" />
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
SNS:
<img width="1903" height="876" alt="Screenshot 2026-03-04 175747" src="https://github.com/user-attachments/assets/03d3b0dc-c047-4404-af67-98edb1a31080" />
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Cloud_watch-Alarm
<img width="1908" height="811" alt="Screenshot 2026-03-04 180413" src="https://github.com/user-attachments/assets/27bb7d8c-48a4-4de8-8c06-03a8f17a3875" />
------------------------------------------------------------------------------------------------------------------------------------------------------------------
Output:
<img width="1919" height="1079" alt="Screenshot 2026-03-04 180437" src="https://github.com/user-attachments/assets/23f1d63b-c10f-484c-9c58-72f83029f351" />

Perfect 👍
Based on your architecture image (VPC + ALB + Private EC2 + ASG + AMI + Launch Template), I’ll now give you:


# 🚀 PROJECT: Highly Available NGINX Web App on AWS

Architecture Flow:

User → ALB (Public Subnet) → Target Group → Private EC2 (NGINX)
Private EC2 → NAT Gateway → Internet (updates)
ASG → Auto scaling using Launch Template

<img width="1350" height="902" alt="Screenshot 2026-02-19 232106" src="https://github.com/user-attachments/assets/1188bcc1-ab28-49bd-9643-aae4528fca39" />

---

# ✅ STEP 1: Create VPC

### Why?

To isolate your network.

Create VPC:

* CIDR: `10.0.0.0/16`

This gives large IP space.

---

# ✅ STEP 2: Create 4 Subnets (2 AZs)

| AZ  | Public      | Private     |
| --- | ----------- | ----------- |
| AZ1 | 10.0.1.0/24 | 10.0.2.0/24 |
| AZ2 | 10.0.3.0/24 | 10.0.4.0/24 |

### Why?

Multi-AZ = High Availability.

---

# ✅ STEP 3: Internet Gateway

* Create IGW
* Attach to VPC

Update Public Route Table:

```
0.0.0.0/0 → IGW
```

Associate with public subnets.

---

# ✅ STEP 4: NAT Gateway (Public Subnet)

Create NAT in Public Subnet (AZ1 recommended per AZ for production).

Allocate Elastic IP.

Update Private Route Table:

```
0.0.0.0/0 → NAT Gateway
```

### Why?

Private instances can install packages securely.

---

# ✅ STEP 5: Launch Bastion Host (Public Ubuntu)

* Public Subnet
* Enable Public IP
* Allow SSH (22)

Used to access private EC2.

---

# ✅ STEP 6: Launch Private EC2 (Ubuntu + NGINX)

* Subnet: Private
* No Public IP
* Security Group:

  * Allow HTTP (80) from ALB SG
  * Allow SSH from Bastion SG

---

# 🔹 STEP 7: Install NGINX on Private Ubuntu

SSH into private EC2 via bastion.

Update system:

```bash
sudo apt update -y
sudo apt upgrade -y
```

Install NGINX:

```bash
sudo apt install nginx -y
```

Start NGINX:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Check status:

```bash
sudo systemctl status nginx
```

---

# 🔹 STEP 8: Deploy Static Website (NGINX)

Default root directory:

```
/var/www/html
```

Replace default page:

```bash
cd /var/www/html
sudo rm index.nginx-debian.html
sudo nano index.html
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
<title>NGINX AWS Project</title>
</head>
<body>
<h1>Welcome to My NGINX Private EC2 🚀</h1>
</body>
</html>
```

Set permissions:

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

Restart:

```bash
sudo systemctl restart nginx
```

---

# ✅ STEP 9: Create Target Group

* Type: Instance
* Protocol: HTTP
* Port: 80
* Health check path: `/`

Register private EC2.

---

# ✅ STEP 10: Create Application Load Balancer

* Internet-facing
* Select Public Subnets (AZ1 + AZ2)
* Listener: HTTP 80
* Attach Target Group

Test using ALB DNS.

---

# ✅ STEP 11: Create AMI

From private EC2 → Create Image.

Why?
To clone configured NGINX server.

---

# ✅ STEP 12: Create Launch Template

Include:

* AMI
* Instance type
* Security group
* Key pair
* User data (optional)

Example user-data (auto-install NGINX):

```bash
#!/bin/bash
apt update -y
apt install nginx -y
systemctl start nginx
systemctl enable nginx
```

---

# ✅ STEP 13: Create Auto Scaling Group

* Select Private Subnets
* Attach Target Group
* Set:

  * Min: 1
  * Desired: 2
  * Max: 4
* Scaling policy:

  * Target tracking (CPU 60%)

---

# 🔥 FINAL FLOW

Client → ALB → Target Group → ASG → Private NGINX EC2
Private EC2 → NAT → Internet

---

# 🧠 ADVANCED INTERVIEW QUESTIONS

---

# 🔹 Networking

### 1️⃣ Why is ALB in public subnet?

Because it must receive internet traffic.

---

### 2️⃣ Why are EC2 instances private?

Security — no direct internet access.

---

### 3️⃣ Why NAT in public subnet?

NAT requires access to IGW.

---

### 4️⃣ How does routing work internally?

Route tables determine traffic direction.

---

# 🔹 NGINX Advanced

### 5️⃣ Where is NGINX config file?

```
/etc/nginx/nginx.conf
```

Virtual host config:

```
/etc/nginx/sites-available/default
```

---

### 6️⃣ How do you test NGINX config?

```bash
sudo nginx -t
```

---

### 7️⃣ How do you reload NGINX without downtime?

```bash
sudo systemctl reload nginx
```

---

### 8️⃣ Difference between restart and reload?

* Restart → stops and starts service
* Reload → applies config without downtime

---

# 🔹 Load Balancer Deep Questions

### 9️⃣ How does health check work?

ALB sends HTTP request to `/`.
If 200 response → healthy.

---

### 🔟 What happens if one AZ fails?

ALB routes traffic to healthy AZ.

---

# 🔹 Auto Scaling Deep

### 11️⃣ How does ASG detect unhealthy instance?

Through ELB health checks + EC2 health checks.

---

### 12️⃣ What scaling policies exist?

* Target tracking
* Step scaling
* Scheduled scaling

---

### 13️⃣ What is lifecycle hook?

Allows custom actions during launch/terminate.

---

# 🔹 Security

### 14️⃣ How do security groups communicate?

Allow inbound from ALB SG only.

---

### 15️⃣ How do you secure SSH?

* Use Bastion
* Disable password auth
* Use key-based login

---

# 🔹 Real Scenario Question

### ❓ Traffic increases suddenly to 5000 requests/sec.

Answer:

* ALB distributes traffic
* ASG scales out
* New instances auto-register
* Unhealthy instances replaced

---

# 🔹 Production Improvements

To make enterprise-grade:

* Enable HTTPS (ACM certificate)
* Add WAF
* Use Multi-AZ NAT
* Enable CloudWatch alarms
* Enable access logs
* Add Route 53 domain
* Use IAM roles instead of static credentials

---

# 🎯 1-Minute Interview Explanation

> I built a highly available AWS architecture using a custom VPC with multi-AZ subnets. I deployed Ubuntu-based private EC2 instances running NGINX to host a web application. An Application Load Balancer distributes traffic to private instances through a target group. NAT Gateway enables secure outbound internet access. I created an AMI and Launch Template to configure an Auto Scaling Group for dynamic scaling and fault tolerance.


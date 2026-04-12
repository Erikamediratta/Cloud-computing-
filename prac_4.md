 
##  Objective

Set up an Auto Scaling Group (ASG) with a launch template, configure scaling policies based on CPU utilization, deploy an Application Load Balancer (ALB) to distribute traffic, and test the setup by simulating high traffic.

---


### Why Auto Scaling?

Without auto scaling, you must manually add servers when traffic spikes and remove them when traffic drops. This leads to either:
- **Over-provisioning** — wasting money running too many idle servers
- **Under-provisioning** — poor performance or downtime during traffic spikes

**Auto Scaling** automatically adjusts the number of EC2 instances in response to demand, ensuring your application has the right capacity at the right time.


A **Launch Template** defines the configuration for every new instance the ASG launches.


1. Go to **EC2 → Launch Templates → Create launch template**
2. Configure:

```
Launch template name:     web-server-temp
Template version:         1 (auto)
Description:              Web server for auto scaling

AMI:                      Amazon Linux 2023 AMI (free tier)
Instance type:            t2.micro
Key pair:                 my-ec2-keypair
Security Group:           Create new OR select existing
  - Inbound: HTTP (80) from 0.0.0.0/0
  - Inbound: SSH (22) from My IP
  - Outbound: All traffic
```

**User Data (bootstrap script) — paste under "Advanced details → User data":**
```bash
#!/bin/bash
# Update packages
yum update -y

# Install Apache web server
yum install -y httpd

# Create a simple webpage that shows instance ID
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)

cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head><title>Auto Scaling Demo</title></head>
<body style="font-family: Arial; text-align: center; padding: 50px;">
    <h1>🚀 Auto Scaling is Working!</h1>
    <p><strong>Instance ID:</strong> ${INSTANCE_ID}</p>
    <p><strong>Availability Zone:</strong> ${AZ}</p>
    <p>Refresh to see load balancing distribute to different instances</p>
</body>
</html>
EOF

# Start and enable Apache
systemctl start httpd
systemctl enable httpd
```

3. Click **Create launch template**

### Step B — Create an Auto Scaling Group

1. Go to **EC2 → Auto Scaling Groups → Create Auto Scaling group**

**Step 1 — Name and launch template:**
```
Auto Scaling group name: web-asg
Launch template:         web-server-template (Version: Latest)
```

**Step 2 — Instance launch options:**
```
VPC:      Select your VPC (or default VPC)
Subnets:  Select at least 2 public subnets in different AZs
          (e.g., us-east-1a, us-east-1b)
```


**Step 3 — Configure advanced options (Load Balancing):**
```
Load balancing: Attach to a new load balancer
  Load balancer type:    Application Load Balancer
  Load balancer name:    web-alb
  Load balancer scheme:  Internet-facing
  
  Listeners and routing:
    Protocol: HTTP  Port: 80
    Default routing: Create a target group
      Target group name: web-target-group

Health checks:
  ✅ Turn on Elastic Load Balancing health checks
  Health check grace period: 300 seconds
```


**Step 4 — Configure group size and scaling:**
```
Group size:
  Desired capacity:  2
  Minimum capacity:  2
  Maximum capacity:  5

Scaling policies:
  Select: Target tracking scaling policy
  Policy name:        scale-on-cpu
  Metric type:        Average CPU utilization
  Target value:       70
  Instance warmup:    300 seconds
<img width="1491" height="605" alt="image" src="https://github.com/user-attachments/assets/f001b2ee-bf40-4c71-b877-a12bc3a93b89" />

```



**Step 5 — Review and Create**

> ⏳ Wait 3–5 minutes for the instances to launch and become healthy

---

### Step C — Deploy Application Load Balancer (Review)

The ALB was created as part of the ASG setup above. Let's verify and understand it.

**Navigate to: EC2 → Load Balancers → web-alb**


```
DNS Name: web-alb-XXXXXXXX.us-east-1.elb.amazonaws.com
State: Active
Availability Zones: us-east-1a, us-east-1b

Listeners:
  HTTP:80 → Forward to web-target-group

Target Group (web-target-group):
  Targets: (your 2+ EC2 instances)
  Health Status: healthy ✅
```
If no healthy, open target groups, register targets, select the running instance with the same VPC as the target group 
and security group must be http 80 add, and also public subnet, user data add

```
cd downloads
ls
chmod 400 first_keypair.pem
ssh -i first_keypair.pem ec2-user@3.231.58.251
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
echo "Hello from EC2" | sudo tee /var/www/html/index.html
```
the load balancer security group http 80 

<img width="1276" height="606" alt="image" src="https://github.com/user-attachments/assets/699427ea-e3ca-4050-9a70-86e9cb624e2f" />


**Access your application:**
```
Open browser: http://web-alb-XXXXXXXX.us-east-1.elb.amazonaws.com
```




the scheme of load balancer must be internet -facing, and dns name must not start with internal

Access application
Open browser: http://web-alb-XXXXXXXX.us-east-1.elb.amazonaws.com

---

### Step D — Test Auto Scaling by Simulating High Traffic

#### Method 1 — CPU Stress Test (from inside the instance)
```bash
# SSH into one of your instances
ssh -i my-ec2-keypair.pem ec2-user@<instance-public-ip>

# Install stress tool
sudo yum install -y stress

# Stress the CPU for 5 minutes (300 seconds)
stress --cpu 4 --timeout 300

# In another terminal, watch CPU usage
top
# or
watch -n 1 'uptime'
```
<img width="960" height="232" alt="image" src="https://github.com/user-attachments/assets/82ca3850-78da-4d86-b774-02e118505699" />



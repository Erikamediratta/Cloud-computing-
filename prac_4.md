 
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
Launch template name:     web_server
Template version:         1 (auto)
Description:              Web server for auto scaling

AMI:                      Amazon Linux 2023 AMI (free tier)
Instance type:            t3.micro
Key pair:                 first_keypair
Security Group:           Create new OR select existing
  - Inbound: HTTP (80) from 0.0.0.0/0
  - Inbound: SSH (22) from My IP
  - Outbound: All traffic
```

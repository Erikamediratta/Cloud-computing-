Objective-
Deploy a virtual machine on AWS using Amazon EC2, configure security groups for SSH access, and connect to the instance remotely via SSH.

Amazon Elastic Compute Cloud (EC2)provides scalable virtual computing capacity in the cloud. Each running EC2 virtual machine is called an instance. You can think of it as renting a computer in Amazon's data center.


From EC2 service, launch instance, configure the name and tags, 	•	
Select Amazon Linux 2023 AMI (or Amazon Linux 2)
* Architecture: 64-bit (x86)
* This is free tier eligible
* 
<img width="878" height="513" alt="image" src="https://github.com/user-attachments/assets/7f33be00-c5c1-4815-8406-aea6f0020afc" />



Instance type-t2.micro  →  1 vCPU, 1 GB RAM  →  Free tier eligible

<img width="862" height="303" alt="image" src="https://github.com/user-attachments/assets/34d63f11-e584-47fc-93ff-e467730aa846" />

Key pair login-.pem- linux/mac
.ppk-windows

<img width="673" height="579" alt="image" src="https://github.com/user-attachments/assets/49e6e873-7201-42e5-8a71-4574feb6d8e7" />


Network Settings

* VPC: default
* Subnet: No preference (any AZ)
* Auto-assign public IP: Enable
* Firewall (Security Groups): Create security group
    * Security group name: my-ec2-sg
    * Allow SSH traffic: from My IP (not 0.0.0.0/0 in production)


<img width="824" height="566" alt="image" src="https://github.com/user-attachments/assets/3a219580-62f3-4e21-bd83-16db676b72a0" />


Configure security groups to allow SSH access

Inbound Rules:
┌──────────┬──────────┬────────┬─────────────────────────┐
│ Protocol │   Port   │ Source │       Description        │
├──────────┼──────────┼────────┼─────────────────────────┤
│   TCP    │    22    │ My IP  │ SSH Access               │
│   TCP    │   80     │ 0.0.0.0│ HTTP (if hosting a site) │
│   TCP    │  443     │ 0.0.0.0│ HTTPS                    │
└──────────┴──────────┴────────┴─────────────────────────┘
Outbound Rules:
│   All    │   All    │ 0.0.0.0│ Allow all outbound       │

To modify after launch

EC2 Dashboard → Instances → Select instance →
Security tab → Security groups → Edit inbound rules

#### CONNECT TO THE INSTANCE USING SSH

EC2 → Instances → Select your instance →
Public IPv4 address: e.g., 54.123.45.67

chmod 400 prac_2_keypair.pem

Ssh command-

ssh -i <path-to-key.pem><username>@<public-ip-or-dns>

<img width="815" height="326" alt="image" src="https://github.com/user-attachments/assets/25b08b28-5250-4e97-843d-5db099472c56" />

You are now connected

#### basic commands to explore the instance

# System info
whoami                    # Shows current user
hostname                  # Shows hostname
cat /etc/os-release       # OS details
uname -r                  # Kernel version

# Hardware resources
nproc                     # Number of CPUs
free -h                   # RAM usage
df -h                     # Disk usage
lscpu                     # CPU details

# Network info
curl ifconfig.me          # Shows public IP
ip addr show              # Shows network interfaces

# Update packages
sudo yum update -y        # Amazon Linux 2
sudo dnf update -y        # Amazon Linux 2023

# Install something
sudo yum install -y httpd  # Install Apache web server
sudo systemctl start httpd
sudo systemctl enable httpd



<img width="755" height="815" alt="image" src="https://github.com/user-attachments/assets/04b4798f-8cce-49ee-8062-b9e373951c3a" />


<img width="810" height="647" alt="image" src="https://github.com/user-attachments/assets/8ed2cdc1-3ec4-4a5f-9504-63dcd66db68f" />


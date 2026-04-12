
Objective: Create and configure a Virtual Private Cloud (VPC).

a) Create a custom VPC with a public and private subnet.

b) Launch an EC2 instance in the public subnet and another in the private subnet.

c) Configure an Internet Gateway for internet access in the public subnet.

d) Use a NAT Gateway to provide internet access for instances in the private subnet.




What is a VPC?
A Virtual Private Cloud (VPC) is a logically isolated section of the AWS cloud where you can launch AWS resources in a network that you define.
It gives you complete control over your virtual networking environment, including IP address ranges, subnets, routing tables, and gateways

#### Public vs private subnet

Public Subnet:
  ✅ Has a route to Internet Gateway (0.0.0.0/0 → IGW)
  ✅ Instances CAN have public IP addresses
  ✅ Reachable from internet (controlled by security groups)
  Use case: Web servers, load balancers, bastion hosts

Private Subnet:
  ❌ NO direct route to Internet Gateway
  ❌ Instances do NOT have public IPs
  ✅ Can reach internet via NAT Gateway (outbound only)
  Use case: Databases, application servers, internal services


  AWS CONSOLE->VPC->YOUR VPCs->CREATE VPC

  <img width="1225" height="620" alt="image" src="https://github.com/user-attachments/assets/05bf76c7-64e0-4862-a48f-38cc97ddb5da" />

  #### CREATE A PUBLIC SUBNET

  <img width="1059" height="542" alt="image" src="https://github.com/user-attachments/assets/f0e0680a-098f-4b25-b967-3e196eb0feec" />


  <img width="1059" height="542" alt="image" src="https://github.com/user-attachments/assets/f0e0680a-098f-4b25-b967-3e196eb0feec" />

  
<img width="1251" height="143" alt="image" src="https://github.com/user-attachments/assets/f0979040-92e1-4736-9e57-a179fec3331e" />


Select prac_3_public → Actions → Edit subnet settings →
Enable auto-assign public IPv4 address → Save


#### CREATE A PRIVATE SUBNET 

<img width="1270" height="79" alt="image" src="https://github.com/user-attachments/assets/bbac6e28-5401-43cf-8560-2fe828717919" />
(do not enable public IP address for this)

#### LAUNCH EC2 IN PUBLIC AND PRIVATE SUBNETS

#### public EC2 instance

Instance name:   public_instance_2
AMI:             Amazon Linux 2
Instance type:   t3.micro
VPC:             prac_3_vpc
Subnet:          prac_3_public
Auto-assign IP:  Enable
Security Group:  Allow SSH (port 22) from My IP, HTTP (80) from anywhere
Key pair:        first_keypair

<img width="1219" height="575" alt="image" src="https://github.com/user-attachments/assets/0d131cb5-69f2-4636-a7bc-2036be8f5cbd" />


#### private EC2 instance

Instance name:   private_instance
AMI:             Amazon Linux 2
Instance type:   t3.micro
VPC:             prac_3_vpc
Subnet:          prac_3_private
Auto-assign IP:  disable
Security Group:  Allow SSH (port 22) from My IP, HTTP (80) from anywhere
Key pair:        first_keypair

<img width="1254" height="553" alt="image" src="https://github.com/user-attachments/assets/c1ff82ea-12b4-4e68-b7e0-d9b7eca24900" />


<img width="1017" height="96" alt="image" src="https://github.com/user-attachments/assets/795fd92c-5c7d-4593-88c5-d945e7077220" />


#### CONFIGURE INTERNET GATEWAY

go to VPCs->internet Gateway->create Gateway

Attach the created Gateway to VPC

select my_gateway->actions-> attach to VPC -> prac_3_vpc ->attach

<img width="1039" height="153" alt="image" src="https://github.com/user-attachments/assets/bc098d97-2a04-44be-8079-dcc9757a26c2" />

#### CREATE A PUBLIC ROUTE TABLE

vpc ->create route table

configure with name and prac_3_vpc

#### Add route to internet gateway 

Select public_rt → Routes tab → Edit routes → Add route:
  Destination: 0.0.0.0/0
  Target:      Internet Gateway → my_gateway
→ Save changes

<img width="1271" height="389" alt="image" src="https://github.com/user-attachments/assets/39a5d4b3-58a6-42f1-8c39-f68a51655539" />


Select public_rt → Subnet associations tab → Edit subnet associations →
prac_3_public → Save associations

Now any instance in public-subnet-1 can reach the internet!

#### CONFIGURE NAT GATEWAY 

 NAT Gateway (Network Address Translation) sits in the public subnet and allows private instances to initiate outbound internet connections (e.g., to download updates), but blocks inbound connections from the internet.

 ALLOCATE AN ELASTIC IP

 VPC → Elastic IPs → Allocate Elastic IP address → Allocate
Note the Allocation ID: 


CREATE NAT GATEWAY

Go to VPC → NAT Gateways → Create NAT gateway
Configure:
Name:               my_natgw
Subnet:           prac_3_public← MUST be in public subnet!
Connectivity type:  Public
Elastic IP:         Select the one you just allocated
Click Create NAT gateway

#### Associate Private Subnet with Private Route Table



#### CREATE A PRIVATE ROUTE TABLE

vpc ->create route table

configure with name and prac_3_vpc

#### ADD ROUTE TO NAT GATEWAY

Select private_rt → Routes → Edit routes → Add route:
  Destination: 0.0.0.0/0
  Target:      NAT Gateway → my_natgw
→ Save changes

Associate Private Subnet with Private Route Table
Select private_rt → Subnet associations → Edit subnet associations →
prac_3_private → Save associations

#### VERIFICATION

✅ 🔹 Verify Public EC2 Internet Access

After launching your instance in Amazon Web Services:

1. Set key permissions
chmod 400 my-ec2-keypair.pem
2. SSH into Public EC2
ssh -i my-ec2-keypair.pem ec2-user@<PUBLIC_EC2_PUBLIC_IP>
3. Test internet access
ping google.com

✔ If working → Public EC2 has internet access

✅ 🔹 Verify Private EC2 Access (via Public EC2 / Bastion)
Step 1: Add key to SSH agent (on your local machine)
ssh-add my-ec2-keypair.pem
Step 2: SSH into Public EC2 with agent forwarding
ssh -A -i my-ec2-keypair.pem ec2-user@<PUBLIC_EC2_PUBLIC_IP>

👉 -A = enables forwarding your key to the next hop

Step 3: SSH into Private EC2 (from Public EC2)
ssh ec2-user@<PRIVATE_EC2_PRIVATE_IP>

✔ If this works → your bastion setup is correct

✅ 🔹 Verify Private EC2 Internet (via NAT Gateway)

Now you are inside the Private EC2:

1. Test connectivity
ping google.com

✔ Should work → means NAT Gateway is configured correctly

2. Check outbound public IP
curl ifconfig.me

👉 Output will show:

NAT Gateway Elastic IP
NOT the private EC2 IP






  

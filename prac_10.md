## 10. Cloud Security
**Objective:** Understand security practices in the cloud.

**a)** Implement IAM roles and policies for a cloud platform.

**b)** Create and assign least-privilege roles to users.

**c)** Configure data encryption for storage (e.g., S3 bucket encryption).

**d)** Set up a firewall rule and test its functionality.


Click IAM->policies->create policy-> choose json and 

```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:ListBucket"],
    "Resource": "arn:aws:s3:::new-bucket-2026-erika/*"
  }]
}
```

<img width="961" height="379" alt="image" src="https://github.com/user-attachments/assets/ba90a5d9-784e-45b7-a302-5d3324df8a27" />

CREATE A ROLE

roles->create role->select trusted entity-> attach the s3ReadOnlyPolicy->name it s3ReadOnlyRole->create role

<img width="1003" height="507" alt="image" src="https://github.com/user-attachments/assets/0c5ba316-ca11-4bd8-acfc-072a00f5eb58" />


#### B) Create and assign least-privilege Roles to Users

Create a user,
Attach minimum permissions->choose attach policies directly, search and attach only what they need (s3ReadOnlyAccess)
set password,in security credentials, enable console access with temporary password and again scroll to access keys,  create access keys for programmatic access only

NOW ASSIGN TO A GROUP, create a group with limited policies and add the user to that group instead of assigning policies directly


<img width="930" height="285" alt="image" src="https://github.com/user-attachments/assets/50c36bf0-4f4d-4157-aeb2-e0f0f0401621" />

<img width="881" height="508" alt="image" src="https://github.com/user-attachments/assets/3a3b1d98-e198-44f1-896d-836a2536cd10" />

 
#### TEST

 <img width="574" height="288" alt="image" src="https://github.com/user-attachments/assets/8af31dbb-3df8-4add-b49c-70a2fd1f28aa" />

**** 
SO, s3 ls(list) is allowed, s3 rm(delete) is denied, because policy does not allow Deleteobject, cp (upload) is also denied because our policy does not allow PutObject

<img width="993" height="520" alt="image" src="https://github.com/user-attachments/assets/94e75076-f992-44ec-ad5d-b564c0fc16a5" />

#### Step C Configure data encryption for s3 bucket 

open s3 bucket( or create one ) , enable default encryption by going into properties tab, scroll to default encryption and edit by selecting
server-side encryption with amazon s3 mmanaged keys (SSE-S3) and save the changes

Now go to permissions tab-> bucket policy->

```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:PutObject",
    "Resource": "arn:aws:s3:::new-bucket-2026-erika/*",
    "Condition": {
      "StringNotEquals": {
        "s3:x-amz-server-side-encryption": "AES256"
      }
    }
  }]
}

```
This means blocks any upload that isn't encrypted


#### Step D Set up a firewall Rule and test it

Create a security group, ADD INBOUND RULES

EC2 → Security Groups → Create security group
```
Name:         webserver-strict-sg
Description:  Strict rules for production web server
VPC:          your-vpc

Inbound rules:
  HTTP    | TCP | 80   | 0.0.0.0/0     | Allow public HTTP
  HTTPS   | TCP | 443  | 0.0.0.0/0     | Allow public HTTPS  
  SSH     | TCP | 22   | 203.0.113.0/24| Allow SSH from office IP range only
  
Outbound rules:
  HTTP    | TCP | 80   | 0.0.0.0/0     | Allow outbound HTTP (updates)
  HTTPS   | TCP | 443  | 0.0.0.0/0     | Allow outbound HTTPS
  DNS     | UDP | 53   | 0.0.0.0/0     | Allow DNS
  (Remove the default "All traffic" outbound rule)
```

<img width="1251" height="383" alt="image" src="https://github.com/user-attachments/assets/349d6da4-9cfc-4925-9806-e691a5c7bc71" />




<img width="1242" height="546" alt="image" src="https://github.com/user-attachments/assets/1a7c97dc-7618-4042-8207-983ced48cc8e" />


This blocks all other ports by default(no extra rules=denied)

ATTACH THIS SECURITY GROUP TO THE RUNNING EC2 INSTANCE


<img width="1049" height="304" alt="image" src="https://github.com/user-attachments/assets/b623c425-7647-46c2-af7b-5038681a5c77" />
****

#### TEST


```
ssh -i first_keypair.pem ec2-user@<public ip>
sudo yum update -y
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```
```
# Test HTTP access (should work if web server running)
curl http://<ec2-public-ip>

# Test SSH from your IP (should work)
ssh -i keypair.pem ec2-user@<ec2-public-ip>

# Test a port that's NOT in the rules (should timeout)
sudo yum install -y nc
nc -zv <ec2-public-ip> 8080
# Connection timed out (blocked by security group)

# Test from unauthorized IP for SSH (should be blocked)
# From a different IP than what's in the rule:
ssh ec2-user@<ec2-public-ip>
# ssh: connect to host X.X.X.X port 22: Connection timed out
```

<img width="1168" height="793" alt="image" src="https://github.com/user-attachments/assets/42983916-b81e-4dad-ad2f-10b34ccddf4c" />







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


#### B) Create and assign ;east-privilege Roles to Users

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




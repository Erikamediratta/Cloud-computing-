
## 9. Set Up Networking
**Objective:** Configure OpenStack Neutron to provide networking for instances.

**a)** Create a private network and a public network.

**b)** Attach a router to connect the private network to the public network.

**c)** Assign floating IPs to instances for external access.


#### Create a priavte network 

project->network->networks->create network

<img width="1190" height="138" alt="image" src="https://github.com/user-attachments/assets/0659df2b-9864-4d14-acfa-92f36f4c0a64" />

#### Create Router and attach it to private network

<img width="1119" height="333" alt="image" src="https://github.com/user-attachments/assets/9b421637-6fdd-4fef-b60c-90cf6a6073dd" />

<img width="1130" height="464" alt="image" src="https://github.com/user-attachments/assets/99b0bdc1-af52-4945-8b12-ce84a7db1817" />


#### Create a VM with private network

Launch instance

<img width="1214" height="274" alt="image" src="https://github.com/user-attachments/assets/c43efd9e-6719-408b-aefd-975239fd2d41" />

#### Floating IPs

<img width="963" height="396" alt="image" src="https://github.com/user-attachments/assets/06697789-c896-4cdf-8b6a-bed77278f9e4" />


<img width="1131" height="424" alt="image" src="https://github.com/user-attachments/assets/f047fc67-3198-4602-9391-cf03dc51068e" />


#### Create security group

<img width="1160" height="362" alt="image" src="https://github.com/user-attachments/assets/0dbd1808-2b0d-4127-9107-3a7922a88847" />

<img width="1064" height="519" alt="image" src="https://github.com/user-attachments/assets/fa890e5f-642f-4098-af8b-4f5e49711802" />




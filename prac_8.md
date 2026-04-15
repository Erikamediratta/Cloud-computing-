<img width="1276" height="714" alt="image" src="https://github.com/user-attachments/assets/c90c416a-c42c-4dac-9a33-f67ca0e75a27" />


<img width="1273" height="299" alt="image" src="https://github.com/user-attachments/assets/3042ad04-1510-4611-8975-c3941b093a0d" />


<img width="1266" height="649" alt="image" src="https://github.com/user-attachments/assets/19c3dac5-915d-4190-817e-2439ea50cbf2" />

#### Create Image

Go to project->compute->image->create image

<img width="1075" height="675" alt="image" src="https://github.com/user-attachments/assets/126a9609-2ee4-477d-bdcb-6c4858535e0b" />

#### Create flavor

Go to Admin->compute->flavor->create flavor

<img width="980" height="663" alt="image" src="https://github.com/user-attachments/assets/48a4f8ca-6efe-4dab-bd70-7b152ec18d25" />

####Launch instance

firstly,install cli by

```
sudo apt install -y python3-openstackclient
source /var/snap/microstack/common/etc/microstack.rc
```

Pre-requisite check

```
# List available images
openstack image list

# List available flavors
openstack flavor list

# List available networks
openstack network list

# Create a key pair for SSH access
openstack keypair create --private-key my-key.pem my-keypair
chmod 400 my-key.pem
```

Project->compute->instance->launch instance


<img width="1143" height="272" alt="image" src="https://github.com/user-attachments/assets/b12a3143-0a2e-46ca-98c5-4ee7fa6f21e6" />



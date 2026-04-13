#### Objective : install Openstack for practice




#### step 1:Update your system

```
sudo apt update && sudo apt upgrade -y
```

#### step 2: install required packages 

```
sudo apt install -y git python3-pip
```

#### step 3: create stack user

```
sudo useradd -s /bin/bash -d /opt/stack -m stack
```

#### give admin access (no password needed)

```
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
```

#### switch to stack user

```
sudo su - stack
```

#### step 4 : download devstack

```
git clone https://opendev.org/openstack/devstack
cd devstack
```

#### step 5 : check branches 

```
git branch -r | grep stable
git checkout master
```

#### step 6: fix logs folder

```
sudo mkdir -p /opt/stack/logs
sudo chown -R stack:stack /opt/stack/logs
```

#### step 7 : find network interface 

```
ip addr
```

 ** Look for something like **

 ```
ens33
eth0
enp0s3
```

 #### step 8 : create config file 

 ```
nano local.conf
```

```
[[local|localrc]]
ADMIN_PASSWORD=secret123
DATABASE_PASSWORD=secret123
RABBIT_PASSWORD=secret123
SERVICE_PASSWORD=secret123

FLOATING_RANGE=192.168.1.224/27
FIXED_RANGE=10.11.12.0/24
FIXED_NETWORK_SIZE=256

FLAT_INTERFACE=ens33   # CHANGE THIS

LOGFILE=/opt/stack/logs/stack.sh.log
VERBOSE=True
LOG_COLOR=True

IMAGE_URLS="http://download.cirros-cloud.net/0.6.2/cirros-0.6.2-x86_64-disk.img"
```

 ctrl+0_->enter


 #### step 9 : run devstack

 ```
./stack.sh
```

#### step 10: 
```
http://YOUR_IP/dashboard
```

#### step 11 : verify 

```
source /opt/stack/openrc admin admin
openstack service list
openstack compute service list
openstack network agent list
openstack image list
```


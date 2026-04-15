#### Objective : install Openstack for practice




#### step 1:Update your system

```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install snapd -y
sudo systemctl enable snapd
```

```
sudo snap install microstack --beta --classic

```
```
sudo microstack init --auto --control
```

```
microstack.openstack credentials
```

OR
```
cat/var/snap/microstack/common/etc/microstack.rc
```
then
```
source /var/snap/microstack/common/etc/microstack.rc
```

Connect via public ip address of the ec2 instance, and if not happening then 
terminal->ssh -i first_keypair.pem -L 8000:localhost:443 ubuntu@public ip

then run in browser->
http://localhost:8000

OR
https://localhost:8000

TEST->
```
microstack.openstack server list
```


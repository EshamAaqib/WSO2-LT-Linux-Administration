# WSO2-Linux-Administration-Assignemt

## 1. Setting the VM name as “prod-devops-master” and setting timezone as “Asia/Colombo”

### Changing VM Name (Execute the following in a terminal)

```
sudo hostnamectl set-hostname prod-devops-master
```

### Then we can run the following to change the timezone

```
sudo timedatectl set-timezone Asia/Colombo
```

## 2. Adding Linux users range from user1 to user7

### We can execute the following to create users from user1 - user7

```
sudo adduser user1
sudo adduser user2
sudo adduser user3
sudo adduser user4
sudo adduser user5
sudo adduser user6
sudo adduser user7
```

## 3. Creating 2 Linux user groups called grpA and grpB and adding the above created users to Linux groups as mentioned in the document.

### To create the two groups execute the following

```
sudo groupadd grpA
sudo groupadd grpB
```

### Finally we can assign the users to the created groups accordingly by executing the following 

```
sudo adduser user1 grpA 
sudo adduser user2 grpA
sudo adduser user3 grpA 
sudo adduser user4 grpB 
sudo adduser user5 grpB
sudo adduser user6 grpB
```
###### If needed we can open the /etc/host file to verify that all the changes were applied as expected.

## 4. Creating a new disk partition using a loopback device, formating it with XFS filesystem type, Mounting it under /mnt and automounting the partition during a reboot

### First lets create a file so we can create a loopback device using it. Execute the following

```
dd if=/dev/zero of=loopbackfile.img bs=100M count=1
```

### Now we can create a loopback device using the above created file. Execute the following

```
sudo losetup -fP loopbackfile.img
```

### To find out the loopback device created execute the following (We will need this when creating a partition so note it down. Mine is /dev/loop12) 

```
losetup -a
```

### Afterwards we can create a partition using the newly created loopback device (The loopback device path will be needed in this step which we noted down in the previous step) 

```
sudo fdisk /dev/loop12
```
###### IMPORTANT : STEP 1: After executing the above you will be prompted with some options. Press n and hit enter to create a new partition. Then press p and hit enter to create a primary partition. Then it will prompt you to enter the partition number, first sector and last sector. Just press enter so it will assign the default values. Finally enter w and write the changes. Again enter fdisk executing the above command and press p to list the newly created partiton path. Note it down as we will need it later. (Mine is /dev/loop12p1)

```
sudo partprobe -s
```
###### STEP 2 : Execute the above to reflect the changes to the kernel




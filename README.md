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

###### IMPORTANT : STEP 1: After executing the below you will be prompted with some options. Press n and hit enter to create a new partition. Then press p and hit enter to create a primary partition. Then it will prompt you to enter the partition number, first sector and last sector. Just press enter so it will assign the default values. Finally enter w and write the changes. Again enter fdisk executing the above command and press p to list the newly created partiton path. Note it down as we will need it later. (Mine is /dev/loop12p1)

```
sudo fdisk /dev/loop12
```

###### STEP 2 : Execute the above to reflect the changes to the kernel

```
sudo partprobe -s
```

###### STEP 3 : Now we need to format the partition using the XFS filesystem. Execute the following to do so

```
sudo mkfs.xfs /dev/loop12p1
```

###### STEP 4 : Now We can mount the newly created and formatted partition to /mnt. Execute the following to do so

```
sudo mount /dev/loop12p1 /mnt
```
###### RUN ```df -hP``` to verify

###### STEP 5 : Finally we should auto mount the partition to /mnt on every reboot. At this point I went ahead and created a shell script and ran it as a systemd service to run on every boot. Here is the shell script that I created and the systemd file

```
#!/bin/bash
#Script to automatically create and mount loopback partition
losetup -fP /home/eshamaaqib/loopbackfile.img
sudo mount /dev/loop12p1 /mnt
```
###### IMPORTANT : After creating the shell script execute ```sudo chmod +x SCRIPTNAME``` to make the script executable

###### Now we can create a .service file in /lib/systemd/system to run the the script on every reboot. I executed ```sudo nano /lib/systemd/system/mountloop.service```. Here is the content of the .service file.

```
[Unit]
Description=Auto create and mount loopback device

[Service]
ExecStart= LOCATIONOFTHESCRIPT/SCRIPTNAME

[Install]
WantedBy=multi-user.target
```
###### Point the ExecStart to your script.

###### Now execute the following to enable and start the service that we created previously.

```
sudo systemctl enable NEWLYCREATEDSERVICENAME.service
sudo systemctl start NEWLYCREATEDSERVICENAME.service
```
###### Finally execute ```sudo systemctl status NEWLYCREATEDSERVICENAME.service``` to check the status of the service. If it outputs succeeded that means everything is configured properly. Now on every reboot it should automatically create the loopback device and mount the partition.

###### Thats it now we have created a loopback device using a file, created a partition using it, formatted it using the XFS filesystem, then mounted it to /mnt and created the loopback device and mounted the partiton on every reboot using a shell script.

## 5. Creating 3 folders inside /mnt. Execute the following to do so

```
sudo mkdir share-grpA
sudo mkdir share-grpB
sudo mkdir share-common
```

## 6. Configuring the folder permissions

### Part A : /mnt/share-grpA -> user1 is the owner . All members in grpA as well as the owner should be able to view files inside the folder and modify or create new files. Execute the following to assign the permissions.

```
sudo chown -R user1 share-grpA 
sudo chgrp grpA ./share-grpA 
sudo chmod 770 share-grpA 
```

### Part B : /mnt/share-grpB -> user4 is the owner . All members in grpB as well as the owner should be able to view files inside the folder and modify or create new files. Execute the following to assign the permissions.

```
sudo chown -R user4 share-grpB  
sudo chgrp grpB ./share-grpB 
sudo chmod 770 share-grpB 
```

### Part C : /mnt/share-common -> user1 is the owner . All members in grpA and grpB should be able to view and create new files inside the folder. But they should not be able to delete another user’s file. Anyone else should not be able to view,create or delete files inside this folder. Execute the following to assign the permissions.

```
sudo chown -R user1 share-common  
sudo setfacl -m g:grpA:rwx share-common 
sudo setfacl -m g:grpB:rwx share-common 
sudo chmod 1770 share-common 
```

## 7. Allowing CTO to only view the files inside the shared folder without adding CTO to grpA or grpB. 

```
sudo setfacl -m u:user7:rx share-grpA 
sudo setfacl -m u:user7:rx share-grpB 
sudo setfacl -m u:user7:rx share-common 
```

## 8.  Installing the git command for cloning Github repositories

```
sudo apt install git
```

## 9. Encrypting /mnt partition using cryptsetup tool

### First we need to unmount the partition

```
sudo umount /mnt
```

### Now we can start to encrypt the partition by executing the following

```
sudo cryptsetup -y -v luksFormat /dev/loop12p1 
```
###### /dev/loop12p1 is the partition we created using the loopback device. Yours might be different. You will be prompted to enter a new password after executing the above.

### Now mount the encrypted partition by executing the following 

```
sudo cryptsetup luksOpen /dev/loop12p1 mnt
```
###### This will not mount it to /mnt. It will be mounted to /dev/mapper/mnt. In order to use it as a partition and mount it to /mnt we will need to format and map it.

### Formatting the partition

```
sudo mkfs.xfs /dev/mapper/mnt
```

### Now we can mount it to /mnt

```
sudo mount /dev/mapper/mnt  /mnt
```

### Finally we should automate it to mount on every reboot and unlock the volume using a key

###### First we need to create a key to unlock the partition wihtout using a password. To do so execute the following

```
dd if=/dev/urandom of=disk_secret_key bs=512 count=8
```
###### After running the above command the key will be stored in your current working directory
```
sudo cryptsetup -v luksAddKey /dev/loop12p1  disk_secret_key
```
###### The above command will prompt you to enter a password. Enter the password you provided when encrypting the parition 
```
sudo cryptsetup -v luksOpen /dev/loop12p1 loop12p1_crypt --key-file= disk_secret_key 
```
###### After running the above command you should be greeted with a message stating "Key slot 1 unlocked" if you configured everything properly 

###### Now as the key is created we can automate the whole process so it will mount on every reboot. I went ahead and modified the shell script that I created in part 4. Here is the modified shell script.

```
#!/bin/bash
#Script to automatically create and mount encrypted loopback partition and map it to /mnt folder
losetup -fP /home/eshamaaqib/loopbackfile.img
cryptsetup -v luksOpen /dev/loop12p1 loop12p1_crypt --key-file=/LOCATIONOFTHEKEY/disk_secret_key
mount /dev/mapper/loop12p1_crypt  /mnt
```
###### Now on every reboot it should automatically mount the partition without requiring a password on every reboot

## 10. Monitoring the shared folder to find and remove files older than 90 days using bash script.

### First I went ahead and created a bash script. Here is the bash script.

```
#!/bin/bash
#Script to delete files in the /mnt folder that is older than 90 days
find /mnt -mtime +90 -type f -delete
```

### Then added the following to the bottom of crontab so it will run everyday at 11:59PM. Crontab can be edited by executing ```sudo crontab -e```.

```
59 23 * * * /LOCATIONOFTHESCRIPT/SCRIPT
```

###### This will check and delete files older than 90 days in the /mnt folder everyday at 11:59PM.




## This whole assignment was written on a private repository on GitHub I used GitFront to share the private repository and here is the link : https://gitfront.io/r/eshamaaqib/6b74ba6b67d59d8a88e5af8c8c904e324401ae92/WSO2-Linux-Administration-Assignment/



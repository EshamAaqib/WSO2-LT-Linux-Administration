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






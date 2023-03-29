MySQL Data Directory Change in AWS Ubuntu

1. Create AWS EC2 with Ubuntu OS
2. Connect to EC by SSH then Install MySQL by command:

    sudo apt update
    sudo apt install mysql-server
    
    Start MySQL by using the below Command: 

    sudo systemctl start mysql.service
3. Create Data Volume & Attache with MySQL installed AWS EC2
4. Mount The Data Volume to EC2 by use the below command to see the new disk
   
    sudo lsblk
    
    ubuntu@ip-10-0-0-230:~$ sudo lsblk
    NAME     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    loop0      7:0    0  24.4M  1 loop /snap/amazon-ssm-agent/6312
    loop1      7:1    0  55.6M  1 loop /snap/core18/2679
    loop2      7:2    0  63.3M  1 loop /snap/core20/1778
    loop3      7:3    0 111.9M  1 loop /snap/lxd/24322
    loop4      7:4    0  49.8M  1 loop /snap/snapd/17950
    xvda     202:0    0     8G  0 disk
    +-xvda1  202:1    0   7.9G  0 part /
    +-xvda14 202:14   0     4M  0 part
    +-xvda15 202:15   0   106M  0 part /boot/efi
    **xvdf     202:80   0   100G  0 disk**

5. Format the Disk by this command 

sudo mkfs -t ext4 /dev/xvdf


6. Create a new directory for mounting the New Volume


 sudo mkdir /mnt/newvolume


7. Now mount the volume by this command 

sudo mount /dev/xvdf /mnt/newvolume


8. Now can see the disk on the list  by command 


df -h

Filesystem      Size  Used Avail Use% Mounted on


**/dev/xvdf        98G   24K   93G   1% /mnt/newvolume**


9. Stop MySQL Service by 


sudo service mysql stop

and Copy the existing data directory to the new location where the new disk mounted 

 sudo cp -R /var/lib/mysql /mnt/newvolume/mysql


10. Change the data directory on 


sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf 


 as below as per the mount disk location. 

. . .
datadir=/mnt/newvolume/mysql
. . .

11. Then change the Directory ownership & permission on the new mount volume as below:
    
    
     ubuntu@ip-10-0-0-230:~$ sudo chown -R mysql:mysql /mnt/newvolume/*
     
     
     ubuntu@ip-10-0-0-230:~$ sudo chmod -R 755 /mnt/newvolume/*

    
12. Edit the APPARMOR file to allow MySQL directory change as 


 sudo vi /etc/apparmor.d/tunables/alias


 Update the alias file by adding the following line at the bottom of the file:

```
alias /var/lib/mysql/ -> /mnt/newvolume/mysql/,

```

Then reload the AppArmor profiles by running the following command:

sudo service apparmor reload



13. Finally, start the MySQL service by running the following command:

sudo service mysql start

Now MySQL data directory has been successfully changed to the new location.

14. Finally, time to Check the changed directory in the Database: 

   To find the temporary user of MySQL run below command;


       sudo cat /etc/mysql/debian.cnf


    Then log in by the temporary user gets on the file:


        mysql -u debian-sys-maint -p


Then run the below query on MySQL==

mysql> select @@datadir;
+-----------------------+
| @@datadir             |
+-----------------------+
| /mnt/newvolume/mysql/ |
+-----------------------+
1 row in set (0.00 sec)
```

15. To ensure that the changes to the MySQL data directory are persistent after a reboot, add the following line to /etc/fstab:

```
/dev/xvdf /mnt/newvolume ext4 defaults,nofail 0 2

```

This will automatically mount the new volume at boot time.

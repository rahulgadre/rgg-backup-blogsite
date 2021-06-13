---
title: Bastion host & Private instances  
date: 2021-06-12 18:58:47 +07:00
# modified: 2020-07-11 16:49:47 +07:00
tags: [aws]
description: Accessing private instances via SSH from the bastion host in AWS
---

Accessing private instances via SSH using a bastion host:

Often times we have instances in private subnets which we want to access using SSH. The most common way of accessing private instances is by using a bastion host. A bastion host is an instance which can be publicly accessed either from your computer or from the outside world. It is then used to access private instances. It is recommended to set the source as MyIP for SSH (port 22) in the security group associated with a bastion host.

Once we have access to the bastion host, the private instances can be accessed using any of the 2 methods described below:

--> Userdata - if the keys for bastion host and private instances are different.

If the private instances were launched using a key different than the one used to deploy the bastion host, then a new SSH key or the bastion host key can be added in the private instances using Userdata. Please note that this method requires stopping and starting the private instance. Therefore, please put the private instance on standby if it's part of an autoscaling group and ensure that the on shutdown action isn't set to terminate. 

1.    Create a new key pair or use the existing bastion host key pair.

2.    If you create the private key in the Amazon EC2 console, then retrieve the public key for the key pair.

      Use the following command to generate a key that will be used in step 6:

      ``` 
      ssh-keygen -y -f mykp.pem --- replace the mykp.pem with the name of your key.
      ```

3.    Open the Amazon EC2 console.

4.    Stop your instance.

5.    Choose Actions, Instance settings, Edit user data.

6.    Copy the following script into the Edit user data dialog box:


``` 

Content-Type: multipart/mixed; boundary="//"
MIME-Version: 1.0

--//
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloud-config.txt"

#cloud-config
cloud_final_modules:
- [users-groups, once]
users:
  - name: username
    ssh-authorized-keys: 
    - PublicKeypair  

```

Note:  Ensure to enter the entire public key generated in step 2 in place of PublicKeypair in the script above , the public key starts with ssh-rsa. Also, be sure to set the right username (ec2-user/centos) in the script above depending upon the OS of your private instance.

The aforementioned userdata script was added in the blog by referring to this [article](https://aws.amazon.com/premiumsupport/knowledge-center/user-data-replace-key-pair-ec2/).

Once the above userdata script has been added, start the private instance and then access the private instance from the bastion host using the following command:

```
ssh -i mykp.pem ec2-user@private_IP
```

Note: Be sure that you have the key used in step 2 on the bastion host to have successful SSH connection with a private instance.

--> Using a private key - if the key for the bastion host and private instances is the same.

If the bastion host and the private instances were launched using the same key, then a private key can be copied from your local computer onto the bastion host. Here are the steps to copy the private key from your local computer onto the bastion host:

1. On your computer, navigate to the folder where the key is located.
2. Use the cat command to view the contents of the key - cat mykp.pem.
3. Copy all the lines of the key displayed on the screen.
4. Access the bastion host and create a new file in the default user's (ec2-user/centos) home folder - vi mypriv.pem.
5. Paste the contents copied in step 3 into the new file created on the bastion host.
6. Save the file by pressing esc followed by :wq.
7. Change the file permissions to 400 by using the chmod command - chmod 400 mypriv.pem.
8. Log into a private instance via SSH using the newly created key (mypriv.pem).

```
ssh -i mypriv.pem ec2-user@private_IP
```
Note: Be sure to replace the keyname mypriv.pem with the name that was selected while creating the key file in step 4.

These are the two commonly used methods to access private instances via a bastion host.


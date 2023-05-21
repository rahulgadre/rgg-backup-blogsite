---
title: Installing an SSM agent locally on an EC2 instance
date: 2023-05-20 20:15:27 +07:00
tags: [aws]
description: This blog contains steps to install an SSM agent locally on an EC2 instance
---

Often times, an EC2 instance is passing both the health checks, but not accessible via SSH. One way to diagnose OS level issues is by detaching the root volume of the impaired instance and attaching it to another instance within the same availibility zone. Another way is install an SSM agent on the impaired instance (if it's not already installed) and inspect the SSH or OS level issues directly at the OS level of the impaired instance.

In this article, we are going to see how to install an SSM agent locally and access the impaired instance via Session Manager for further troubleshooting. Once the root volume of the impaired instance is attached to a rescue instance within the same availibility zone, execute the following commands for SSM agent installation:

```
$ mount /dev/<mount> /mnt
$ wget https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm  -P /mnt/home
$ for i in dev run sys proc ;do mount --bind /$i /mnt/$i;done
$ chroot /mnt
$ rpm -ivh /home/amazon-ssm-agent.rpm
$ systemctl enable amazon-ssm-agent
$ exit
$ for i in dev run sys proc ;do umount  /mnt/$i;done
$ umount /mnt
```

After running the above commands successfully, unmount and detach the root volume of the impaired instance from the helper instance. Next, attach it to the impaired instance with the device label  as /dev/xvda or /dev/sda1 depending upon the AMI. After starting the impaired instance, verify if the instance is now accessible via Session Manager.

##### Note: The above steps were successfully run and tested on a RHEL 8.2 instance. 

#### Troubleshooting

After completing the above procedure, the impaired instance should be accessible via Session Manager. However, if the instance isn't accessible via [Session Manager](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/session-manager.html) due to some reason, then the following list can be checked to assist with fixing the issue. Please note that the list described below only references a few commonly observed issues and further troubleshooting may be required if the Session Manager access issue persists.

- Confirm the [prerequisites](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started.html) for enabling Session Manager.
- Check if the instance has internet connectivity or VPC endpoints for connection to SSM endpoints.
- Perform the attach/detach method again and check if SELINUX is enabled (this may prevent auto start of SSM agent on boot)
- Inspect /var/log/messages depending upon the distribution of operating system for further understanding of issues related to starting an SSM agent. 

Enable root access on an EC2 Linux instance:

Although it is a best practice to have access to EC2 Linux instances via username & password or session manager or EC2 instance connect, but SSH access with the root user can be configured. This blog describes the steps to enable and access EC2 instances with root level access.

1. Connect to the EC2 instance via SSH with the default username and password.
2. Edit the /etc/ssh/sshd_config file.
3. Update the following lines - 

   ```	
   PasswordAuthentication yes
   PermitRootLogin yes
   ```
4. Save the file.
5. Run the appropriate following command to restart the ssh service as per the OS configuration and Linux distribution.
   ```
   sudo service sshd restart
   sudo systemctl restart sshd
   ```
6. Next, update the cloud-init configuration file /etc/cloud/cloud.cfg -

   ```
   disable_root: false
   ssh_pwauth:   true
   ```

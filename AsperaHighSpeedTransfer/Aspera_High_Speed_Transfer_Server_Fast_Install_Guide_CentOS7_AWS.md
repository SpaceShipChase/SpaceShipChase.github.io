# Fast and Dirty Aspera High Speed Transer Installation on CentOS7
This and other guides avaialbe on my website:
https://spaceshipchase.github.io/

Want to try Aspera in your environment quickly? This guide gives the commands
for a very basic installation. It is not meant for a produciton install. It
is not very secure. Please read and understand all the commands before you
execute them. You are responsible for your own system.


This guide was written for installation on CentOS7,
but works well with some tweaks on AWS linux.
There are some differences between CentOS and most
AWS linux images, like firewall and users, but the important commands are the same.

## Step 1 - Installer and License
Obtain the Aspera High Speed Transfer Server (HSTS) installer and license from
your IBM account or from an IBM technical sales representative (me).

## Step 2 - VM Requirements
Able into your VM with a user that has root access.

Your VM needs to be accessible on the network via IP or FQDN.

At least 2 cores and 4 GM or RAM.

Fast network interface (this will likely be the limiting factor for yor transfer speeds).

8GB+ Enough storage to store the files you want to transfer. This is dependent on your needs.
You can also point the HSTS to store data in S3 or another storage option. Guide on that to come soon.




## Step 3 - Configure SSH access, firewall, change SSH port, disable SELinux, restart.

Firewall and ssh config.
```console
systemctl enable firewalld
systemctl start firewalld
firewall-cmd --permanent --add-port=33001/tcp
firewall-cmd --permanent --add-port=33001/udp
firewall-cmd --reload
firewall-cmd --list-all
sed -i 's/#Port 22/Port 33001/' /etc/ssh/sshd_config
sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
systemctl restart sshd.service
```
Check to make sure ssh is listeing on port 33001.
```console
>ss -l | grep 33001
```

Disable SELinux and restart (this is a quick and dirty install).
```console
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/selinux/config
sudo shutdown -r now
```




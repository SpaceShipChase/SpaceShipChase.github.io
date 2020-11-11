# Fast and Dirty Aspera High Speed Transfer Server Installation on CentOS7

Want to try Aspera in your environment quickly? This guide gives the commands
for a very basic installation. It is not meant for a produciton install. It
is not very secure. Please read and understand all the commands before you
execute them. You are responsible for your own system.


This guide was written for installation on CentOS7,
but works well with some tweaks on AWS linux.
There are some differences between CentOS and most
AWS linux images, like firewall and users, but the important commands are the same.

## Step 1 - Installer and License.
Obtain the Aspera High Speed Transfer Server (HSTS) installer and license from
your IBM account or from an IBM technical sales representative (me).

## Step 2 - VM Requirements.
Able to SSH into your VM with a user that has root access.

Your VM needs to be accessible on the network via IP or FQDN.

At least 2 cores and 4 GM or RAM.

Fast network interface (this will likely be the limiting factor for yor transfer speeds).

8GB+ enough storage to store the files you want to transfer. This is dependent on your needs.
You can also point the HSTS to store data in S3 or another storage option. Guide on that to come soon.




## Step 3 - Configure SSH access, firewall, change SSH port, disable SELinux, restart.
Note: SELinux is not a barrier to Aspera.

SSH into your VM.

Firewall and ssh config:
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
Dobuble check to make sure ssh is listeing on port 33001:
```console
ss -l | grep 33001
```

Disable SELinux and restart (this is a quick and dirty install):
```console
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/selinux/config
sudo shutdown -r now
```

Verify SELinux status, and firewall config:
```console
sestatus
firewall-cmd --list-all
```

## Step 4 - Install and load the license.

Transfer the installer and the license to the VM. I like scp.

Put the installer anywhere convenient for you.

Put the license file in the same directory, you will be moving it after the install.

My commands looked like the below:
```console
scp -P 33001 /<source file path> <your_user>@<IP>:/<file destination>
```

SSH back into the VM and install.

>NOTE: Some systems may not have two perl pre requisites. Check and see if
they are on your system and install them if they are not there:
>```console
>yum install perl-Digest-MD5
>yum install perl-Data-Dumper
>```

Install:
```console
yum --nogpgcheck install /<path to installer>
```

Move the license into the /opt/aspera/etc directory and name it aspera-license:
```console
cp /<path to your license> /opt/aspera/etc/aspera-license
```

Load the license:
```console
ascp -A
```

You will see you license number and some server info displayed. Good job so far.


## Step 5 - Prepare for Watch Folders - They are cool and you'll want to play with them anyways.

Create Node API User - this creates a user for the HSTS Node API, which is used for Watch Folders.
For simplicity, I used node_ibmuser/node_password, but you should not use that. It also associates the
API user with the user that will be running your transfers (most likely the user you are using to install
this with in this installation). The last part of the commands sets the ACL for that user.

```console
/opt/aspera/bin/asnodeadmin -a -u node_ibmuser -p node_password -x <your_user> --acl-set admin,impersonation
```
Check it and make sure it worked correctly:
```console
sudo /opt/aspera/bin/asnodeadmin -l
```
You will see output similar to:
```console
List of Node API user(s):
user          system/transfer       user acls
============= ===================== =====================
node_ibmuser  root                  [admin,impersonation]
```

Enable Services for Watch Folders and set the user for them:
```console
systemctl enable NetworkManager
systemctl enable NetworkManager-wait-online.service
/opt/aspera/sbin/asperawatchd --user <your_user>
/opt/aspera/sbin/asperawatchfolderd --user <your_user>
systemctl restart asperarund
```


## Step 6 - Set the server name and docroot.
The docroot is the location the HSTS uses as it's base to store the files you transfer.
The below commands name the server, create the doc root, set the HSTS to use it, set the,
transfer user's permissions on it, then reset the aspera services.

```console
asconfigurator -x "set_server_data;server_name,169.62.1.18"
mkdir /AsperaDocRoot
asconfigurator -x "set_node_data;absolute,/AsperaDocRoot"
asconfigurator -x "set_user_data;user_name,root;read_allowed,true"
asconfigurator -x "set_user_data;user_name,root;write_allowed,true"
asconfigurator -x "set_user_data;user_name,root;dir_allowed,true"
asconfigurator -x "set_user_data;user_name,root;absolute,/AsperaDocRoot"

systemctl restart asperacentral
systemctl restart asperanoded
```

## Step 7 - Create a conneciton to the server and try a transfer!
You are done with the install.

Go to one of your Aspera endpoints or another Aspera server and attempt a transfer.










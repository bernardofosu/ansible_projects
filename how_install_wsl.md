# How to Install Windows Subsystem for Linux (WSl)
## System Requirements to install wsl
[Go to this website to check system reuirements]()

## Steps to Install Windows Subsystem for Linux (WSl)
- Search for Turn Windows Features On and Off
- Turn on Virtual Machine Platform
- Turn on Windows Subsystem for Linux (WSl)
- Click ok and After its done Restart 

## After Retart, Open Command Prompt
### Check the wsl status
```bash
wsl --status
```

#### Success Response
```bash
Default Version: 2

The Windows Subsystem for Linux kernel can be manually updated with 'wsl --update', but automatic updates cannot occur due to your system settings.
To receive automatic kernel updates, please enable the Windows Update setting: 'Receive updates for other Microsoft products when you update Windows'.
For more information please visit https://aka.ms/wsl2kernel.

The WSL 2 kernel file is not found. To update or restore the kernel please run 'wsl --update'.
```

## How to update or restore the kernel 

```bash
wsl --update
```

## How to change default version to 2
```bash
wsl --set-default-version 2
```

## Go to Microsoft Store and Install Ubuntu

## How to list all you installed packages
```bash
sudo apt list --installed
```

### How to check ssh version
```bash
ssh -V
```

### How to install openssh-server
```bash
 sudo apt update && sudo apt upgrade
 sudo apt install openssh-server
```

## How ssh to another wsl server
```bash
ssh 172.31.34.168
```
### Response 
```bash
ssh: connect to host 172.31.34.168 port 22: Connection refused
```

### How to configure new ssh port for each wsl server
Since the host machine is using port 22 none of the wsl server can use it because it is in use
```bash
sudo nano /etc/ssh/sshd_config
```
```bash
Port 2226
#AddressFamily any
ListenAddress 0.0.0.0
#ListenAddress ::
```
### After that reboot
```bash
sudo reboot
```
NB: When you configure port number for one wsl server others cannot also use it

### You have to connect remotely by specifying the port number
```bash
 ssh -p 2226 server3@172.31.34.168
```

## How to check if the status of ssh or sshd
```bash
root@DESKTOP-J7M3OKM:~# systemctl status sshd
```
### Response
```bash
Unit sshd.service could not be found.

The message "Unit sshd.service could not be found" indicates that the SSH server (sshd) might not be installed on your system, or it might be managed under a different service name.
```
### Difference between systemctl status ssh and systemctl status sshd 
The commands systemctl status ssh and systemctl status sshd are used to check the status of the SSH service on your system. The correct command to use depends on your Linux distribution and how the SSH service is named within its systemd configuration.

#### Understanding the Service Names (SSH):
ssh.service: In some distributions, particularly Debian (Ubuntu, Kali Linux) and its derivatives like Ubuntu, the SSH service is named ssh. Therefore, to check its status, you would use:

```bash
systemctl status ssh
```

#### Understanding the Service Names (SSHD):
sshd.service: In other distributions, such as Red Hat-based (AlmaLinux) systems (e.g., CentOS, Fedora) and some versions of openSUSE, the service is named sshd, referring to the SSH daemon. In these cases, you would use:

```bash
systemctl status sshd
```
### Why the Difference?
- The naming convention varies due to historical and organizational preferences:
- Debian and Derivatives: These systems often name the service ssh, aligning with the protocol name. However, they also provide an alias for sshd.service to maintain compatibility. This means that on such systems, both commands might work interchangeably. 
UNIX STACK EXCHANGE
- Red Hat and Others: These distributions typically use sshd to clearly indicate the daemon responsible for handling SSH connections.
- Determining the Correct Service Name on Your System:
- To identify the appropriate service name for SSH on your system, you can list all active services and search for SSH-related entries:

```bash
systemctl list-units --type=service | grep ssh
```
This command will display any services related to SSH, helping you determine whether ssh.service or sshd.service is in use.

#### Practical Implications:
Starting/Stopping Services: When managing the SSH service (e.g., starting, stopping, restarting), ensure you use the correct service name:


#### For systems using ssh.service
```bash
sudo systemctl restart ssh
```
#### For systems using sshd.service
```bash
sudo systemctl restart sshd
```
NB: Avoiding Errors: Using the incorrect service name may result in errors like "Unit ssh.service not found" or "Unit sshd.service not found." Therefore, it's essential to use the service name specific to your system.

### How to solve these  on the kali linux
```bash
systemctl status ssh
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```
```bash
 sudo reboot
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
Failed to talk to init daemon: Host is down
```
Create a wsl.conf at /etc
```bash
sudo nano /etc/wsl.conf

Content of wsl.conf
[boot]
systemd = true
```
[Watch this video](https://www.youtube.com/watch?v=IuJa6bltNgU)
[visit this website](https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/)

How to check the state of wsl
```bash
wsl -l -v
```

Shutdown the System on Powershell or command prompt
```bash
wsl -t kali-linux
```

How to start it again
```bash
wsl -d kali-linux
```
### After Reboot
```bash
sudo reboot
```

### How solve this eror
```bash
systemctl start ssh
```
### Error Message
```bash
Failed to execute /usr/bin/pkttyagent: No such file or directory
Failed to start ssh.service: Access denied
See system logs and 'systemctl status ssh.service' for details.
```
### How Permission issues: Failed to start ssh.service: Access denied
- use sudo systemctl start ssh insted of systemctl start ssh


# How to solve the error below
```b
systemctl start ssh
```
```bash
Failed to start ssh.service: Interactive authentication required.
See system logs and 'systemctl status ssh.service' for details.
```
```bash
server3@DESKTOP-J7M3OKM:~$ systemctl start sshd
```
```bash
Failed to start sshd.service: Interactive authentication required.
See system logs and 'systemctl status sshd.service' for details.
```
```bash
server3@DESKTOP-J7M3OKM:~$ systemctl status sshd.service
Unit sshd.service could not be found.
```
## By just adding sudo to the systemctl start ssh
```bash
server3@DESKTOP-J7M3OKM:~$ sudo systemctl start ssh
[sudo] password for server3:
````
### Then After starting it, You can check the status
```bash
server3@DESKTOP-J7M3OKM:~$ sudo systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
     Active: active (running) since Sun 2025-01-12 01:12:30 GMT; 13s ago
TriggeredBy: ● ssh.socket
```

## Configure Passwordless Sudo (Optional):
- To avoid interactive authentication issues when starting services like SSH, you can configure passwordless sudo for your user:

### Replace your_username with your actual username:
```bash
echo "your_username ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/your_username
```

To generate all the requested SSH host keys and configuration files, here's how you can create them manually. We'll generate the host keys (ssh_host_rsa_key, ssh_host_ecdsa_key, and ssh_host_ed25519_key), their public counterparts (e.g., ssh_host_rsa_key.pub), and the associated SSH configuration files (ssh_config, sshd_config, etc.).

## Generate SSH Host Keys
RSA Key (for SSH host):

bash
Copy
sudo ssh-keygen -t rsa -b 4096 -f /etc/ssh/ssh_host_rsa_key
ECDSA Key (for SSH host):

bash
Copy
sudo ssh-keygen -t ecdsa -b 521 -f /etc/ssh/ssh_host_ecdsa_key
ED25519 Key (for SSH host):

bash
Copy
sudo ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key
These commands will generate the private keys (ssh_host_rsa_key, ssh_host_ecdsa_key, ssh_host_ed25519_key) and their corresponding public keys (e.g., ssh_host_rsa_key.pub, ssh_host_ecdsa_key.pub, ssh_host_ed25519_key.pub) in the /etc/ssh/ directory.

Step 2: Ensure Permissions are Correct
After generating the keys, ensure that the files have the correct permissions:

bash
Copy
sudo chown root:root /etc/ssh/ssh_host_*
sudo chmod 600 /etc/ssh/ssh_host_*
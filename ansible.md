# How to configure WSls or VMs for Ansible
[How to configure for Ansible]()

[How to configure Cloud VMs for Ansible]()

# How to configure SSH
[How to configure SSH]()

# How to Setup git and Github
[How to Setup git and Github]()

# What is Ansible 
Ansible is an open-source automation tool used for configuration management, application deployment, and task automation. It enables system administrators and DevOps teams to automate repetitive tasks, manage configurations, and deploy software across many systems. Ansible operates without agents, meaning it uses SSH (or WinRM for Windows) to communicate directly with systems and execute tasks.

## What is Ad-Hoc command
An Ad-Hoc command in Ansible is a simple, one-time command used to perform a task or run an operation on one or more systems. Ad-hoc commands allow you to quickly perform simple tasks without writing a full playbook.

Syntax of an Ad-Hoc Command:
```bash
ansible <host-pattern> -m <module> -a "<arguments>"
ansible all -m ping
```

## How to install ansible on Ubuntu
```bash
sudo apt update && sudo apt upgrade
sudo apt install ansible
```

# Important Vs Code Extensions for Wsl and Ansible
- Ansible
- WSL
- Remote - SSH: Editing Configuration Files
- Remote - SSH

# WSL Servers
## Debian Distribution
- server1 (main workstation with ansible) = ssh port number (2222)
- server2 = ssh port number (2223)
- server3 = ssh port number (2226)
- server4 = ssh port number (2229)
- server5 = ssh port number (2227)
- kali linux = ssh port number (2228)

## Red Hat / CentOS Distribution
- almalinux = ssh port number (2225)

# How to create inventory file
You have create the inventory file without extension and put all your hosts inside, it can be IP Address or DNS Name
```bash
sudo nano inventory
```

## You put all your hosts names in the inventory file
```bash
172.31.34.168
```
### If you are using wsl, do it this instead
```bash
[wsls]
wsl2 ansible_host=172.31.34.168 ansible_port=2223 ansible_user=server2
```

## How to set up ssh key
```bash
ssh-keygen -t ed25519 -C "Ben default"
```
passphrase = ben

## How to copy the public key to the servers
```bash
sudo ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2223 server2@172.31.34.168
```
NB: We can copy the public key and paste it under ~/.ssh/authorized_keys

## Now we  can log in using ssh key-pair instead of password
```bash
ssh -i ansible server2@172.31.34.168
ssh: connect to host 172.31.34.168 port 22: Connection refused
```
NB: Port 22 is in use but the host machine so we will configure /etc/sshd_config file for the wsl servers to use different ssh port number

## How to configure new ssh port for each wsl server, Skip when using VMs or Cloud Platforms
Since the host machine is using port 22 none of the wsl server can use it because it is in use
```bash
sudo nano /etc/ssh/sshd_config
```
```bash
Port 2223
#AddressFamily any
ListenAddress 0.0.0.0
#ListenAddress ::
```

# Verify SSH Configuration on the Server:
Ensure that the SSH daemon on the server is configured to allow public key authentication. Check the /etc/ssh/sshd_config file for the following settings:
```bash
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```
## After making any changes, restart the SSH service:
### How to Check the Service Name
Use systemctl to list the available SSH services and confirm the correct name on the linux distribution
```bash
sudo systemctl list-units --type=service | grep ssh
```
#### Success Message
```bash
ssh.service                                    loaded active running OpenBSD Secure Shell server # for debian and ubuntu

sshd.service                                          loaded active running OpenSSH server daemon # for CentOS
```
```bash
sudo systemctl restart sshd
sudo systemctl restart ssh
```

## After making any changes and restarting the SSH service, also reboot the wsl
```bash
sudo reboot
```

## How to check if there is connection
```bash
nc -zv 172.31.34.168 2226 # individaully
```
```bash
ss -tuln | grep ':2223\|:2226\|:2228\|:2225' # group
```

## Now we  can log in using ssh key-pair instead of password with specified ssh port number
```bash
ssh -i ansible -p 2223 server2@172.31.34.168
```

## How to use the ping module to check the connection between the workstation and the servers (client)
```bash
ansible all --key-file ~/.ssh/ansible -i inventory -m ping
```
NB: The -m is the flag that allow to specify module name and ping is the module, different from the ping use to internet connection by pinning ip addresses
## If you get the error below, it can be that you server is off or has stopped or the ssh is i not active
```bash
wsl2 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: connect to host 172.31.34.168 port 2223: Connection refused",
    "unreachable": true
}
```

### Success Message
```bash
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## Inventory file
```bash
[wsls]  
wsl_first_instance ansible_host=172.31.34.168 ansible_port=2223 ansible_user=server2  
wsl_second_instance ansible_host=172.31.34.168 ansible_port=2226 ansible_user=server3
wsl_third_instance ansible_host=172.31.34.168 ansible_port=2228 ansible_user=kali
wsl_fourth_instance ansible_host=172.31.34.168 ansible_port=2225 ansible_user=almalinux
```
- This tells Ansible that, wsl_first_instance is located at IP 172.31.34.168, accessible on port 2223, and you should log in using the server2 user.
- NB: ansible_host, ansible_port, and ansible_user are pre-built variables used by Ansible to define how to connect to remote systems. These variables help Ansible manage remote hosts in your inventory and specify connection details.

### Explanation:
**[wsls]:** Group name "wsls" — can group multiple hosts here.
- This is a group name in the Ansible inventory.
Everything listed underneath [wsls] will be part of this group. In this case, wsl_first_instance is the host under this group.
- Ansible allows you to group multiple hosts under a label (e.g., [servers], [databases], [webservers], etc.) to run tasks collectively on all members of that group.

**wsl_first_instance** (Host alias — identifier for this specific host)
- This is a host name or alias for the machine you're managing. 

**ansible_host=172.31.34.168:** With the Ansible inventory configuration, you don't need to change the actual machine's name or hostname on each individual system you are managing. Instead, you use an alias or a custom name in the inventory file to represent the machine.
- This defines the IP address (or DNS name) of the host you want to manage.
- In this case, 172.31.34.168 is the IP address of wsl_first_instance, and Ansible will use this to connect to the machine during task execution.
- This is useful if the hostname (wsl_first_instance) does not match the system’s actual address or if you want to use an IP address explicitly.

**ansible_port=2223:** This specifies the SSH port that Ansible will use to connect to the host.
By default, SSH uses port 22, but in this case, you're using port 2223 for SSH.
If the system you're managing is set up to listen on a different SSH port (in this case, 2223), you specify this so Ansible knows which port to connect to.

**ansible_user=server2:** This specifies the SSH user to use when connecting to the host.
- In this case, server2 is the username that will be used to SSH into wsl_first_instance.
- It's important that the user has appropriate privileges and access to perform the necessary tasks on the machine. If you’re using key-based authentication, the server2 user should have the corresponding public key added to their ~/.ssh/authorized_keys.

## How to set up ansible config file to shortened our Ad-hoc command
```bash
sudo nano ansible.cfg
```
NB: Ansible config file is run when you run ansible command

### What to put in the ansible.cfg file
```bash
[defaults]
inventory = inventory
private_key_file = ~/.ssh/ansible
```
NB: The ansible.cfg should be in your ansible project working directory 
NB: You might have ansible.cfg at /etc/ansible, We will ignored it because the one we have created manually will override that one.

### After Creating the ansible.cfg file when shortened our ansible run command
```bash
ansible all -m ping
```
Now we are not mannualy specifying the private key and inventory file to use but since we have the config file it will check from there. 
NB: Previous command: **ansible all --key-file ~/.ssh/ansible -i inventory -m ping**


#### How to specify the private key in the inventory file
```bash
wsl_first_instance ansible_host=172.31.34.168 ansible_port=2223 ansible_user=server2 ansible_ssh_private_key_file=~/.ssh/ansible
wsl_second_instance ansible_host=172.31.34.168 ansible_port=2226 ansible_user=server3 ansible_ssh_private_key_file=~/.ssh/ansible
wsl_third_instance ansible_host=172.31.34.168 ansible_port=2228 ansible_user=kali ansible_ssh_private_key_file=~/.ssh/ansible
wsl_fourth_instance ansible_host=172.31.34.168 ansible_port=2225 ansible_user=almalinux ansible_ssh_private_key_file=~/.ssh/ansible
```
```bash
 ansible all -i inventory -m ping
```

#### How to ping individual servers
```bash
ansible wsl_first_instance -m ping
```

## How to list all hosts
```bash
ansible all --list-hosts
```

## How to use ansible command to get information about all your hosts operating system and more
```bash
ansible all -m gather_facts
```

### How to limit to only one server or hosts
```bash
ansible all -m gather_facts --limit wsl_first_instance
```
The inventory is already set up with the name wsl_first_instance for the host with 172.31.34.168 at port 2223, and user server2.
This can be use for troubleshooting idividual servers

## How to change hostname of the individual wsl server (optional)
By default, they all have the same hosts name and ip addresses
```bash
sudo nano /etc/hostname
sudo reboot
```

## How to remove sudo password
[link to that guide](remove_sudo_promt.md)

## How to solve the error where only one host show and the rest give error
```bash
server1@DESKTOP-J7M3OKM:~/ansible_projects$ ansible all -m ping
Enter passphrase for key '/home/server1/.ssh/ansible':
wsl_second_instance | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}


wsl_fourth_instance | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: almalinux@172.31.34.168: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).",
    "unreachable": true
}


wsl_third_instance | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: kali@172.31.34.168: Permission denied (publickey,password).",
    "unreachable": true
}


wsl_first_instance | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: server2@172.31.34.168: Permission denied (publickey,password).",
    "unreachable": true
}
```

## The issue might come from the paraphrase caching from the ssh key pair or cache individually
We can create a ssh key pair without paraphrase or cache it individually with the command bellow
```bash
ansible all -m ping -f 1
```
```bash
Enter passphrase for key '/home/server1/.ssh/ansible':
wsl_first_instance | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
Enter passphrase for key '/home/server1/.ssh/ansible':
wsl_second_instance | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
Enter passphrase for key '/home/server1/.ssh/ansible':
wsl_third_instance | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
Enter passphrase for key '/home/server1/.ssh/ansible':
wsl_fourth_instance | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
```

### The reason ansible all -m ping -f 1 works without issues is due to the following:
**1. Sequential Execution**: The **-f 1** option limits Ansible to execute tasks one host at a time, rather than concurrently. This sequential execution avoids potential conflicts or race conditions when multiple SSH connections are initiated simultaneously, especially when the SSH key passphrase needs to be entered multiple times.

**2. Passphrase Prompting:** When tasks are executed one by one, Ansible prompts for the SSH key passphrase for each host individually. This ensures the passphrase is correctly used for each connection without relying on caching or multiplexing.

**3. Avoidance of SSH Key Reuse Issues:** Running tasks in parallel (without -f 1) could cause Ansible to reuse the SSH key improperly or not handle passphrase prompts efficiently, leading to permission errors. Sequential execution eliminates this problem as each connection gets a fresh passphrase prompt.

**4. Ansible's SSH Handling**: Ansible might struggle with managing concurrent SSH sessions involving the same SSH key with a passphrase, leading to some connections failing if the passphrase isn't correctly cached or reused. By reducing concurrency, this issue is mitigated.
Solution Going Forward:

## How avoid repeatedly entering the passphrase and  ansible all -m ping without -f 1
You can use ssh-agent to cache the passphrase or configure SSH multiplexing, as mentioned earlier. This will allow you to use ansible all -m ping without -f 1 and avoid passphrase prompts for each host.

## How to configure ssh-agent
To configure ssh-agent for caching your SSH key passphrase so you don't have to enter it repeatedly, follow these steps:

**1. Start ssh-agent:** First, you need to start the ssh-agent process, which will manage your SSH keys in the background.
```bash
eval "$(ssh-agent -s)"
```
This command will start the agent and set up the necessary environment variables in your current shell session.
#### Success Response
```bash
Agent pid 2058
```
**2. Add Your SSH Key to the Agent:** Now, add your SSH private key to the ssh-agent. You will need to enter the passphrase for the key the first time you add it.
```bash
ssh-add ~/.ssh/ansible
```
If your SSH key is in a different location or has a different name, adjust the path accordingly. After entering the passphrase once, it will be cached by the agent.

#### Success Response
```bash
Enter passphrase for /home/server1/.ssh/ansible:
Identity added: /home/server1/.ssh/ansible (server1@DESKTOP-J7M3OKM)
```

**3. Verify the Key is Added:** You can verify that your key has been added to the agent by running:
```bash
ssh-add -l
```
This should list your SSH key, showing its fingerprint.
```bash
256 SHA256:74Vfv/Pbweqh8P1y17M/w1oNlYOKw5JKtZkJEc5Ff10 server1@DESKTOP-J7M3OKM (ED25519)
```

## Configure the Agent to Start Automatically
If you'd like ssh-agent to start automatically whenever you open a new terminal session, you can add the following to your shell's initialization file (e.g., ~/.bashrc, ~/.zshrc depending on your shell):

```bash
# Start ssh-agent if not already running
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)"
fi
```
After adding this, run:
```bash
source ~/.bashrc
```
This will ensure that ssh-agent is started automatically for new terminal sessions.

### Use ssh-agent with Ansible
Now that your SSH key is loaded into ssh-agent, Ansible will use this cached key without requiring you to enter the passphrase for each connection.

You can test this by running an Ansible command, and it should not prompt you for the passphrase:
```bash
ansible all -m ping
```
Success Message
```bash
ansible all -m ping
wsl_fourth_instance | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
wsl_third_instance | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
wsl_first_instance | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
wsl_second_instance | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

### Stop the Agent (Optional)
If you no longer need the agent running, you can stop it using:
```bash
ssh-agent -k
```

# Running Elavated Ad-Hoc Commands
Anything on the linux sever that makes changes to the sever itself is going to require to be a root user or have sudo previllages

```bash
ansible all -m apt -a update_cache=true
```
This command will failed because its needs root user or sudo previllages to execute, same as the command below without sudo

### Error messages from the above command 
```bash
wsl_first_instance | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Failed to lock apt for exclusive operation: Failed to lock directory /var/lib/apt/lists/: E:Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)"
}
```
### Similar Linux command that will lead to above error
```bash
apt update
```
### Error messages from the above command 
Reading package lists... Done
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)

NB: The above comparisms are used to know the importance --become --ask-become-pass

## How to add sudo previllages or command
```bash
ansible all -m apt -a update_cache=true --become --ask-become-pass
```

The command **ansible all -m apt -a update_cache=true --become --ask-become-pass** is an Ansible ad-hoc command that updates the package cache on all targeted machines using the apt module. Here's a breakdown of each part:

#### Command Breakdown
**ansible all**
- **ansible:** This is the Ansible command-line tool.

- **all:** This refers to all hosts defined in the inventory. You can replace all with a specific group or hostname to target specific machines.
  
**-m apt**
- **-m:** This flag specifies the module to be used.
- **apt:** The module being used, which is specific to Debian-based systems (like Ubuntu) for managing apt packages.

**-a update_cache=true**
- **-a:** This flag allows you to pass arguments to the module.
  
- **update_cache=true:** This argument tells the apt module to update the package cache (apt update equivalent)., similar to running sudo apt update on a Debian-based system.

**--become**
This flag elevates the command to run with superuser (root) privileges, which is often necessary for package management.

**--ask-become-pass:**
Ansible will prompt you for the sudo password.
ansible all -m apt -a update_cache=true --become --ask-become-pass

### The prompt for the sudo password  (BECOME password:) is for the client (target) servers, not the server running Ansible (control node). 

#### Here's how it works:
- **Control Node (Ansible Server):** This is where you run the Ansible command.
- **Target Nodes (Clients):** These are the remote servers listed in your inventory that Ansible connects to and manages.

##### Prompt Behavior:
When you use **--ask-become-pass**, Ansible will prompt you to enter the sudo password for the user on the target nodes.

The password you enter is used to gain elevated privileges (via sudo) on the target nodes to perform tasks that require root access, such as updating the package cache.

## If the target servers have different sudo passwords, you can handle this in several ways:

You can run the Ansible command or playbook separately for each group of servers that share the same sudo password, using --ask-become-pass each time.

```bash
ansible group1 -m apt -a update_cache=true --become --ask-become-pass
ansible group2 -m apt -a update_cache=true --become --ask-become-pass
```
1. Use --limit to Target Specific Hosts:
Limit the playbook or command to specific hosts with a shared password.
```bash
ansible all -m apt -a update_cache=true --become --ask-become-pass --limit host1,host2
```

## Some useful ansible built in variables
- **ansible_host:** IP or hostname of the target machine.
- **ansible_port:** Port for SSH.
- **ansible_user:** SSH user.
- **ansible_ssh_private_key_file:** Path to the private key.
- **ansible_ssh_common_args:** Additional SSH arguments.
- **ansible_ssh_extra_args:** More SSH arguments.
- **ansible_connection:** Type of connection (e.g., ssh, local, winrm).
- **ansible_ssh_user:** SSH user for authentication.
- **ansible_become:** Whether to escalate privileges.
- **ansible_become_user:** The user to become after privilege escalation.


# Writting Ansible Playbook using YAML
```bash
---

- hosts: all
  become: true
  tasks:
    - name: Install apache on all servers
      apt:
        name: apache2
```

## Command to run the ansible Playbook
```bash
ansible-playbook --ask-become-pass name_of_the_playbook.yaml
```

### Using Random package name to get our playbook failed
```bash
---

- hosts: all
  become: true
  tasks:
    - name: Install apache on all servers
      apt:
        name: nana
```

### How to Update the Repository on Linux
```bash
---

- hosts: all
  become: true
  tasks:
    - name: Update Cache on all the Servers
      apt:
        update_cache: yes
```

### How to install php for apache
```bash
---

- hosts: all
  become: true
  tasks:
    - name: Installing php for apache
      apt:
        name: libapache2-mod-php
```

## How to remove packages by changing the state to absent
```bash
---

- hosts: all
  become: true
  tasks:
  - name: Update Cache on all the Servers
    apt:
        update_cache: yes

  - name: Install apache on all servers
    apt:
        name: apache2
        state: absent
```

## How to use the when condition to do segregation
The when condition is very important when you want to install packges on multiple Distribution

### How to check your Distribution
```bash
cat /etc/os-release
lsb_release -a
```
#### When you try to use the apt module to install packages on Centos or Red Hat, you will get error say no apt
```bash
which apt
which apt-get
```
#### Example 
```bash
- hosts: all
  become: true
  tasks:
  - name: Update Cache on all the Servers
    apt:
        update_cache: yes
    when: ansible_distribution == "Ubuntu"
```

### You can use list for multiple Distribution
```bash
- hosts: all
  become: true
  tasks:
  - name: Update Cache on all the Servers
    apt:
        update_cache: yes
    when: ansible_distribution in ["Ubuntu", "Debian"]
```

#### Using the gather facts ad-hoc command, we will the some in built variables (keys) and values to run the when command
```bash
ansible all -m gatther_facts --limit hostnmane/ip_address/alias_name
```
#### We can use the grep command to know specific things like our OS Name or We can grep any key from gather_facts output
```bash
ansible all -m gatther_facts --limit hostnmane/ip_address/alias_name | grep ansible_distribution
```

### How to use and & or operators
```bash
- hosts: all
  become: true
  tasks:
  - name: Update Cache on all the Servers
    apt:
        update_cache: yes
    when: ansible_distribution == "Ubuntu" and ansible_distribution_version == 8.2
```

### Playbook Configuration for CentOS or Red Hat Distribution
- Playbook side for Red Hat and CentOS, 
- Package Manager = dnf
- Apache2 = httpd
- libapache2-mod-php = php
```bash
- hosts: all
  become: true
  tasks:
  - name: Update Cache on all the Servers
    apt:
        update_cache: yes
    when: ansible_distribution == "CentOS"

  - name: Install apache on all servers
    apt:
        name: httpd
        state: latest
    when: ansible_distribution == "CentOS"

  - name: Installing php for apache
    apt:
        name: php
        state: latest
    when: ansible_distribution == "CentOS"
```

One problem is, on CentOS, the Apache2(httpd) will not start automatically after the installation and we have to also set the port for apache2 (port = 80).

On Amazon Linux 2, dnf is not the default package manager. The default package manager for Amazon Linux 2 is yum (Yellowdog Updater, Modified), which is based on rpm packages, and dnf is typically used in Fedora-based distributions (and later versions of RHEL/CentOS).

However, if you've tried installing Apache using dnf, there could be potential issues related to dependencies, package management conflicts, or configuration files that don't align well with Amazon Linux 2's environment. This could lead to issues with missing unit files, incorrect configurations, or incomplete installations, which might have caused the problems you've encountered.

We can manually do it
```bash
sudo systemctl start httpd # start apache2

sudo firewall-cmd --add-port=80/tcp # allow port 80
```

#### We want to automate everything with ansible 

i have to say i really love ansible and i want learn more 

### How to check Amazon linux features
```bash
ansible all -m gather_facts --limit 34.228.66.28 | grep ansible_distribution
```
```bash
https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
        "ansible_distribution": "Amazon",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/os-release",
        "ansible_distribution_file_variety": "Amazon",
        "ansible_distribution_major_version": "2023",
        "ansible_distribution_minor_version": "NA",
        "ansible_distribution_release": "NA",
        "ansible_distribution_version": "2023",
```

## How to target specific nodes in the inventory
```bash
[all]
54.198.104.165 ansible_user=ubuntu ansible_ssh_private_key_file=~/Desktop/Splunk_Key/splunk_key.pem

54.81.15.114 ansible_user=ec2-user ansible_ssh_private_key_file=~/Desktop/Splunk_Key/splunk_key.pem

[webservers]
54.198.104.165 ansible_user=ubuntu ansible_ssh_private_key_file=~/Desktop/Splunk_Key/splunk_key.pem


[dbservers]
54.81.15.114 ansible_user=ec2-user ansible_ssh_private_key_file=~/Desktop/Splunk_Key/splunk_key.pem
```


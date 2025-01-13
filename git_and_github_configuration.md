# How to set up git on VM to push projects to github

- You have to install git on your VM/EC2
- You can create a project folder on your vm or create a github respository and clone to your vm

NB: You can clone with HTTPS but with SSH you will get permission error

### Clone with HTTPS
```bash
git clone https://github.com/bernardofosu/ansible_tutorials.git
```
#### Response
```bash
Cloning into 'ansible_tutorials'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (6/6), done.
```

### Clone with SSH
```bash
ubuntu@ip-172-31-35-227:~/.ssh$ git clone git@github.com:bernardofosu/ansible_tutorials.git
```
### Response Error
```bash
Cloning into 'ansible_tutorials'...
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

## After Cloning with HTTPS and you want to push back to github
```bash
git init
git status
git add filename
git commit -m "commit message"
git push origin master
```
### Response Error
```bash
Username for 'https://github.com': Blogtech1
Password for 'https://Blogtech1@github.com':
remote: Support for password authentication was removed on August 13, 2021.
remote: Please see https://docs.github.com/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls for information on currently recommended modes of authentication.
fatal: Authentication failed for 'https://github.com/bernardofosu/ansible_tutorials.git/
```

## How to set up ssh permission to overcome the above error
### Create SSH Keygen Pair
```bash
ssh-keygen
```
### Response
```bash
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_ed25519
Your public key has been saved in /home/ubuntu/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:UoWUoiFTb6GcRiFmybXrr4dGkJsnnpGQRPz41SsikAQ ubuntu@ip-172-31-35-227
The key's randomart image is:
+--[ED25519 256]--+
|Eo++=....o.      |
| *=+.=..o.       |
|ooo+*o+..        |
|=.oo.+ o         |
|...=o . S        |
| .*+o. o         |
| ..*o..          |
|  o o..          |
|   ..o.          |
+----[SHA256]-----+
```

### Navigate to .ssh
```bash
cd ~/.ssh/
```
```bash
ubuntu@ip-172-31-35-227:~/.ssh$ ls
ansible      authorized_keys  id_ed25519.pub
ansible.pub  id_ed25519       known_hosts
```
### Open the the content of the SSH Public key
```bash
ubuntu@ip-172-31-35-227:~/.ssh$ cat id_ed25519.pub

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPjVSMcgNAQV4/n9hvC/KD0RTMF4pl2KW4z62aixzkDZ ubuntu@ip-172-31-35-227
```

### Login to your github account
- Click on your profile picture 
- Go to Settings
- Select SSh and GPG Keys
- Add New ssh key
- Give it a Title and paste the public key content at Key

### How to check if there connection or the authentication is successful 
```bash
ssh git@github.com
```
NB: the above command at your server or vm terminal
### Success Response 
```bash
ssh git@github.com
PTY allocation request failed on channel 0
Hi bernardofosu! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```
### Error Response 
```bash
ssh git@github.com
git@github.com: Permission denied (publickey).
```

## Final and Most important step, You can set up your email and username to be able to push to github or Set Up the remote  URL
### NB: With the server with only the CLI, You have to Clone the Repository First to Establish the connection
- Now you need to update your Git remote URL to use SSH instead of HTTPS so you can push to your repository without using a password.
Steps to Fix and Push Your Changes:
- Update Your Remote URL to Use SSH: Run the following command to change the remote URL from HTTPS to SSH:

```bash
git remote set-url origin git@github.com:bernardofosu/ansible_tutorials.git
```
### Verify the Updated Remote URL: Confirm that the remote URL is now set to SSH:
```bash
git remote -v
```
#### Output should look like this:
```bash
origin  git@github.com:bernardofosu/ansible_tutorials.git (fetch)
origin  git@github.com:bernardofosu/ansible_tutorials.git (push)
```
## We can configure our email and username to be able to push to github
```bash
git config --global -user.name "Blogtech1"
```
```bash
git config --global -user.email "ofosubernard@848@gmail.com"
```
### How to check the content of your git config
```bash
cat .git/config
```

### Push Your Changes: Now push your changes using:
```bash
git push origin master
```
This should work without requiring a username or password, as it uses your SSH key for authentication.

## How to check the content of your git config
```bash
cat .git/config
```

## You can also clone with ssh
```bash
ubuntu@ip-172-31-35-227:~/ansible_tutorials$ git clone git@github.com:bernardofosu/ansible_tutorials.git
Cloning into 'ansible_tutorials'...
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 9 (delta 0), reused 9 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (9/9), done.
```


# How to solve the  src refspec master does not match any
```bash
server1@DESKTOP-J7M3OKM:~/ansible_projects$ git push -u origin master
```
## Error Message
```bash
error: src refspec master does not match any
error: failed to push some refs to 'github.com:bernardofosu/ansible_projects.git'
```
## Solution to the above error
```bash
server1@DESKTOP-J7M3OKM:~/ansible_projects$ git push -u origin main
Enter passphrase for key '/home/server1/.ssh/id_ed25519':
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
```
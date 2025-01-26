The error sudo: firewall-cmd: command not found indicates that the firewalld service and its associated firewall-cmd command are not installed on your system.

On CentOS (or RHEL-based distributions), firewalld is often used to manage firewall rules, but it may not be installed by default on all systems.
To resolve this issue, follow these steps:

1. Install firewalld

If you want to use firewalld to manage firewall rules, you will need to install it first.

Run the following command to install firewalld:

sudo yum install firewalld

After installation, start and enable firewalld to make sure it is running:

sudo systemctl start firewalld
sudo systemctl enable firewalld

2. Add Port 80 to the Firewall

Once firewalld is installed and running, you can add port 80 (HTTP) to the firewall with the following command:

sudo firewall-cmd --add-port=80/tcp --permanent

The --permanent option ensures that the rule persists after a reboot.

After adding the rule, reload the firewall to apply the changes:

sudo firewall-cmd --reload

3. Verify the Firewall Rule

To verify that the rule has been added correctly, you can list the open ports with:

sudo firewall-cmd --list-ports

Alternative: Use iptables

If you're not using firewalld and are relying on iptables for firewall management, you can add the rule for port 80 manually with iptables. Here's how you can do it:

    Add the rule:

sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

Save the iptables rules to persist after a reboot:

sudo service iptables save

Restart iptables to apply the rule:

    sudo systemctl restart iptables

4. Check for Existing Firewall Management Tool

To determine whether you're using firewalld, iptables, or another tool to manage the firewall, you can check the status of firewalld:

sudo systemctl status firewalld

If firewalld is not installed, then it’s likely that iptables or another firewall tool is being used. If you are using iptables, the firewall-cmd command will not work, and you'll need to manage the firewall with iptables commands instead.

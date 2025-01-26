apt install apache2 apache2-doc apache2-utils (apache installation)

systemctl status apache2
systemctl enable apache2 (if auto restart disabled for some reason)
systemctl start apache2
systemctl restart apache2
systemctl reload apache2 (more graceful than restart, but not always an option)

apt search libapache2-mod (list all apache modules possible for installation)
apt install libapache2-mod-python (example python module installation)
apt install libapache2-mod-perl2 (example perl)

a2enmod (enable an installed module)
a2dismod (disable an enabled module)

name based virtual hosts (multiple websites on the same server)
a2ensite (enable a web site)
a2dissite (disable a web site)

/etc/apache2/sites-available/"yourwebsite.conf": config files for a given virtual host
create directories listed in "yourwebsite.conf" (apache doesn't create directories)
mkdir -p /full/path/listed/in/conf

a2ensite
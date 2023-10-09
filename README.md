# DevOpsGuide

Detailed guide for setting up a project on a VPS.

## Guide Use

The guide can be split into two sections: Server Setup and Application Setup. 
Server Setup includes the tools and configuration needed for my server and any application it may host. 
Application Setup includes the tools and configuration needed for my test application to run.
An in-depth guide for the development of the application itself can be found [here](https://github.com/ruffineli77/Portfolio-Portal).

This guide will eventually be turned into a shell script to automate my workflow.
I want to take extra time to understand what I'm doing before getting into automation for DevOps.
Until I take the time to learn a more full-featured editor for my server, I'll be using nano to make quick edits to my files.

## Preparation

Make sure you include these steps or something similar: 

- A VPS running Debian 12/Bookworm with root access and IPv4 and IPv6 addresses.
- A domain name purchased to easily find your site when it is on the Web (I include a section for configuring DNS records).

## Server Setup

### 1. Setup SSH Keys

1. Generate the ssh keys.
2. Copy the public SSH key to the server.
3. Connect to the server.

 ```bash
 ssh-keygen -t ed25519 -C "my.email" -b 4096 -f ~/.ssh/vpsKey1
 ssh-copy-id root@my-servers-public-ip-address
 ssh root@my-servers-public-ip-address
 ```

The first step of all is setting up SSH keys to securely access my server.
Debian 12 comes with ssh pre-installed, so installation is not needed typically.
The first command generates the private/public ssh key pair, vpsKey1 and vpsKey1.pub. 
The keys are usually stored in the ~/.ssh folder. 
Running the ssh-keygen command will create the ~/.ssh folder if it doesn't already exist
The email argument works like a comment to show who generated the ssh keys.
The second command uses another OpenSSH utility to move the public key to my public server.
The last command connects to my server using root login and the new ssh keys.
When I configured my VPS, I used a password to create my root user, so password login is enabled automatically on my server. 
From now on, I'll be generating private SSH keys for each public system I have and use those to log in.
This eliminates the security risks that arise if someone manages to get access to my root password or any individual server.

### 2. Update the System

1. Update and upgrade the server.

 ```bash
 sudo apt update && apt full-upgrade
 ```

Its good practice to update a new Unix-like system because new vulnerabilities in the Unix/Linux kernel appear frequently. 
Keeping an updated system ensures these vulnerabilities are addressed whenever possible.

### 3A. Create an Everyday User

1. Create a user.
2. Grant superuser privileges.
3. Copy the public SSH key to the new user's .ssh folder.

 ```bash
 adduser my-new-username
 usermod -aG sudo my-new-username
 ssh-copy-id my-new-username@my-servers-public-ip-address
 ```

Now I create a new user with a different password than my root account.
This new user is given superuser privileges,
so it can perform restricted commands without granting it complete control over my system.
There are two methods of copying my SSH keys to the new user. 
I'll be using the simpler method that uses the ssh-copy-id command.
It automatically copies the ssh key from my root users .ssh folder to my new user's folder
and sets the proper permissions and ownerships.
In situations where I need more control over the process, I can manually copy the SSH keys from my 
root users .ssh folder to my new user's folder.
If network restrictions or an incorrect SSH configuration prevent the ssh-copy-id command from running successfully, 
I'll have to use the manual method.

### 3B. Manually Copying an SSH Key

Refer to this section to manually copy the SSH keys.

1. Create an .ssh directory for my new user
2. Copy the root `.ssh/authorized_keys` file to the new user's `.ssh` folder.
3. Change the owner of the `.ssh` folder and its contents to the new user.
4. Give the user permission to read and write the `.ssh/authorized_keys` file.
5. Set restrictive permissions on the new user's `.ssh` directory.

```bash
 sudo mkdir /home/my-user-name/.ssh
 sudo cp /root/.ssh/authorized_keys /home/my-user-name/.ssh/authorized_keys
 sudo chown $USER:$USER /home/my-user-name/.ssh -R
 sudo chmod 600 /home/my-user-name/.ssh/authorized_keys
 sudo chmod 700 /home/my-user-name/.ssh
```

When I first log into the server via SSH instead of using a root password, a public key already exists in my root account's `.ssh` folder. 
Therefore, password login is not automatically enabled. 
To enable login for my new user, I need to copy the `authorized_keys` file from the root account to my new user account.
The new user is also given the appropriate permissions and ownership, similar to the root account.

### 4. Set up the Firewall (UFW).

1. Display the list of allowed connections.
2. Add a new Allow rule for my new ssh port, port 80 for my websites HTTP traffic, and 443 for my websites HTTPS traffic.
3. Enable the firewall.
4. Display the status of the firewall.
5. Connect to the server from a new terminal.

 ```bash
 sudo ufw status
 sudo ufw allow my-new-port-number && sudo ufw allow 80 && sudo ufw allow 443
 sudo ufw deny 22
 sudo ufw enable
 sudo ufw status
 ssh my-new-username@my-servers-public-ip-address
 ```

This sets my firewall to block allow the new port I'll use for SSH, HTTP, and HTTPS. I check the firewall list, 
allow the new port number, deny the default Openssh port, enable the ssh service and check for errors before 
connecting to the server using my new port number and username. I open up a new terminal window instead of exiting my 
current SSH connection in the same window in case I made a mistake and am unable to log back in. If UFW shows no 
errors, I can move on. If I am unable to connect or get any errors, I can revert the changes I made. From now on, if any 
additional services require access to the firewall, I have to add a new Allow rule.


### 5. Limit SSH Login

1. Make a backup of the config file.
2. Open and edit the SSH configuration file.
3. Restrict connections to a single uncommon port number.
4. Make the server disconnect if a login attempt takes longer than 60 seconds to occur.
5. Disable root login.
6. Public key login is automatically enabled on Debian, so I don't have to uncomment that line.
7. Disable password login.
8. Prevent empty passwords from being accepted during log-in.
9. Set my user as the only one that can access the server.
10. Set my group as the only one that can access the server.
11. Save and exit the sshd_config file.
12. Reload and check the status of the SSHD service and make sure there are no errors.
I Look for ```Active: active (running)``` followed by the date and time that my SSHD service was started. 
This tells me that my SSHD service is running.
If the sshd service fails to start, then the ```Active``` header would look like this ```Active: failed (Result: exit-code)```
Any error messages related to why my service did not start, will be shown at the end of the screen.

  ```bash
 sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
 sudo nano /etc/ssh/sshd_config
 sudo systemctl reload sshd 
 sudo systemctl status sshd
  ```

After running these commands, be sure to check the SSH service status. 
Look for indications that the service is "active (running)" to ensure everything is functioning as expected.

After making the changes,
and the sshd service is running successfully, it's important to confirm that I can still access my public server. 
I open a new terminal window and try to connect to my public server with a new port number. 
If the connection fails, I can revert to the original sshd_config file and correct the errors. 
This is as simple as removing the edited sshd_config file
and copying the backup file to the same location with the original file name.

By disabling the less secure login options and switching to a non-default SSH port, I've significantly reduced the surface area for potential attacks. 
Public key authentication is one of the most secure methods
of accessing my server because, assuming my local system is secure,
I am the only one with the credentials that public server will be looking for.
However, you must securely store your SSH key pairs because they are now the only way to access my public server.
The new port number you specified is also the only one that can access my public server.

In this step, I disable the less secure login options and change the default SSH port for my public server. This means 
using my ssh key pair is the only way to log in to the server. If either key is destroyed, a new pair must be 
generated. Finally, I reload and check the status of the SSH service to make sure it still works.

This is an example of the sshd_config file with enhanced security.

```bash
Include /etc/ssh/sshd_config.d/*.conf

AllowUsers my-user-name
AllowGroups my-group-name

Port my-new-port-number
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

LoginGraceTime  60
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

# PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem	sftp	/usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#	X11Forwarding no
#	AllowTcpForwarding no
#	PermitTTY no
#	ForceCommand cvs server
```

### 6. Prepare the System for my Application

1. Install required system packages. 
2. Install snap.
3. Update snap.
4. Install Certbot.
5. Check the status of newly installed services.

 ```bash
 sudo apt install git python3-pip python3-venv nginx gunicorn clamav snapd
 snap install core; snap refresh core
 snap install --classic certbot
 sudo systemctl status nginx
 sudo systemctl status fail2ban
 ```

At this point, my application is secure enough to start updating, upgrading, and installing packages. Debian 12 comes 
preinstalled with python3, openssh-server, ufw and other packages I may need in the future. 
Using a virtual environment for my applications helps keep python packages local to my project and prevents conflicts. 
This means I can have multiple sets of packages and modules for the same project which makes changing and testing much easier.
This is a quick overview of the packages I installed and why I need each one.

- git. Version control system to manage the changes I make to my code.
- python3-pip and python3-venv. Pythons package manager along a virtual development environment package handle installing and updating the tools that my application needs.
- gunicorn and nginx. A wsgi along with a powerful web server and reverse proxy optimize my application for the Web and provide a protective layer between the sensitive information in my app and the Web.
- clamav
- Fail2Ban is a utility that will scan my log file and ban any IPs that show malicious activity such as too many password failures or seeking exploits.
- Clamav an anti virus/malware service.
- snapd and certbot. Linux's snap daemon. It manages the apps and dependencies that certbot need to work.

## Application Setup

These sections go over the steps needed to build, test, and deploy my application. 

### 7. Prepare the Application

1. Create an application directory.
2. Change ownership of the html folder to my user. [^1]
3. Set permissions the application directory.
4. Initialize a git repository.
5. Clone my project.
6. Create a virtual environment.
7. Activate the environment
8. Install the required packages.
9. Run the test suite and ensure everything works.
10. Deactivate the virtual environment.

     ```bash
    sudo mkdir -p /var/www/my_project
    sudo chown -R $USER:$USER /var/www/my_project
    sudo chmod -R 755 /var/www/my_project
    git init
    git clone https://github.com/my-username/my-project.git
    python3 -m venv my_project_venv
    source my_project_venv/bin/
    python3 -m pip install -r requirements.txt
    python3 wsgi.py
    gunicorn wsgi:test_wsgi -b 0.0.0.0:80
    deactivate
     ```

In this section, I add my application files to a directory I own and add my application files. 
I use a virtual environment and a requirements.txt to keep my installed packages separate from the global system ones.
I test my app with python and then with gunicorn.
The environment is deactivated, so I can install global packages again.

Testing here is important because applications that work flawlessly in a local environment are run through web server and web server gateway interfaces.
These tools are useful and expose problems that arise when using absolute paths and many other bugs I haven't encountered yet.
After all bugs are fixed to documented I can move on to the next step.

Gunicorn recommends that you run it with the system package installed with apt rather than the python package?






















### 8. Set up the Docker container

1. C 
2. T 

    ```bash
   ec
   ec
   ec
    ```
   
Docker handles creating and activating the python virtual environment.
I thhhhinkkk Docker runs the gunicorn instance inside a contained section of the server that only has the tools and applications that my application needs to run.
























### 9. Set up the Web Server (NGINX)

1. Stop any pre-installed web server service.
2. Verify the nginx installation.
3. Check the configuration file for errors. 
4. Stop and restart any nginx services then check the status of the service.

    ```bash
   sudo mkdir
   sudo nginx -v
   sudo nginx -t
   sudo systemctl stop nginx
   sudo systemctl start nginx
   sudo systemctl status nginx
    ```

Nginx is my production web server. 
It stores, process, and delivers the information in my application to the web.  


### 10. Domain Registration

Now, I will connect my server to a domain name. This allows users to access my website without needing to type the 
entire IP address. I'll associate a hostname that corresponds to my server's IPv4 address with an 'A' record, and my 
site's domain name to my server's IPv6 address with an 'AAAA' record.

1. Sign in to your DNS provider's control panel and register a domain name.
If you don't have a DNS provider, some popular options include Namecheap, AWS route 53, and Google Domains.
2. Create two ```A``` records:
For your root domain host, use @. 
For your subdomain host, use www. 
For both records, include your server's public IPv4 address.
3. Create two AAAA records:
Use the same details as for the ```A``` records, but include your server's public IPv6 address instead.
4. Wait for the DNS changes to propagate.
This can take from a few minutes to up to 48 hours.
After this, open your domain name in a web browser. 
5. If you see the Nginx welcome page, it means your DNS setup is correct and the web server is accessible through the 
domain name.

### 11. Configure Server Blocks for Each Website

1. Create a server block for my domain and application to use.
2. Enable the server block by creating a symbolic links between
3. Prevent a possible hash bucket memory problem.
4. Ensure there are no errors in any nginx files.
5. Restart nginx to enable the changes.

   ```bash
   sudo echo '
   server { 
        listen 80;
        listen [::]:80;

        root /var/www/my_project;
        index index.html index.htm index.nginx-debian.html;

        server_name my-domain.com www.my-domain.com;

        location / {
                try_files $uri $uri/ =404;
        }
   }'
   > /etc/nginx/sites-available/my_project
   sudo ln -s /etc/nginx/sites-available/my_project /etc/nginx/sites-enabled/
   sudo nano /etc/nginx/nginx.conf
   sudo nginx -t
   sudo systemctl restart nginx
    ```

Now I set up server blocks to allow my websites to be managed from one web server.
This lets me keep configuration details for each of my domains separate.
The chmod command changes permissions for the owner of a file or directory, all users in the same primary group as
the owner, and for everyone else included unauthenticated and anonymous users.
When I set 600 for the .ssh folder, I allow the owner which is my user to read and write.
Other users and unauthenticated users cannot read, write, or execute commands.

Change this section to point to the Docker container running the gunicorn instance instead of a gunicorn instance. ???





















### 12. Configure SSL Certificates

1. Get the SSL certificate 

    ```bash
   certbot --nginx -d mydomain.name
    ```

CertBot will secure our website with a free ssl cert. 
Having a valid SSL cert allows the site to use HTTP an improved more secure protocol than HTTP. 
Nginx comes with a plugin that handles reconfiguring and reloading the config when necessary.

### 13. Configure Automated Deployment

1. Create a systemd service file to automatically run applications on when Ubuntu starts.
2. Start my service.
3. Enable my service to start when the system powers on.
4. Check the status of the service and make sure there are no errors.

    ```bash
   sudo echo 
   '
   [Unit]
   Description= Project Description
   After=network.target

   [Service]
   User=my-username
   Group=www-data
   WorkingDirectory=/var/www/my_project
   Environment="PATH=/var/www/my_project/project_venv/bin"
   ExecStart=/var/www/my_project/project_venv/bin/gunicorn --workers 3 --bind unix:project_name.sock -m 007 wsgi:app

   [Install]
   WantedBy=multi-user.target
   '
   > /systemd/system/my_project.service
   sudo systemctl start my_project.service
   sudo systemctl enable my_project.service
   sudo systemctl status my_project.service

    ```

Now that my application is ready I can set my server to run my app on when the system starts.

Change this to start the docker container on system start.



















## Conclusion

1. SSH keys are used to securely access my public server.
2. The system is updated to maintain its security.
3. A user with sudo privileges is created for everyday tasks.
4. A firewall is enabled to prevent unwanted or unnecessary network traffic 
5. All other SSH logins options are disabled.
6. System packages the server needs to function are installed.
7. My application is copied to the server and tested with Python and Gunicorn.
8. The Docker container to store my application is created.
9. Nginx is configured to serve my application files.
10. My application is connected to a domain name, so I can access it more easily.
11. A section of my web sever is designated to my test application specifically. 
12. An SSL certificate is generated in order to encrypt the data accessed by a users web browser.
13. A systemd service is created to ??? start my Docker container whenever my system starts or restarts.

In conclusion, this guide outlines a comprehensive approach to deploying a Python application on a public server, 
using best practices for security and maintainability; from setting up SSH keys and system updates to leveraging Docker, Nginx, and SSL encryption. 
Once mastered, I plan to automate this workflow through a shell script and augment it with additional features such as SSH key rotation, monitoring, and backups for future deployments.

## Notes 

unix permissions 

[^1]: The chmod command changes permissions for the owner of a file or directory, all users in the same primary group as
the owner, and for everyone else included unauthenticated and anonymous users.
When I set 600 for the .ssh folder I allow the owner which is my user to read and write.
Other users and unauthenticated users cannot read, write, or execute commands.

## Sources 

Securing Debian - https://www.debian.org/doc/manuals/securing-debian-manual/
Nginx Config Structure - https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts

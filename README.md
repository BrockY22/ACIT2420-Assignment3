# Welcome to my tutorial to create a regular user and install nginx.
## Prerequisite: You have just started from a Fresh Debian 12 server on digitalocean.

# Step 1: Create a new regular user
Replace <newuser> with username of your choice
```bash
useradd -ms /bin/bash <newuser>
```
-m will ensure that the home directory of the new user is created.
-s/bin/bash specifies the login shell for the new user to be bash.


# Step 2: Grant the new user Administrator Privileges.
```bash
usermod -aG sudo <newuser>
```
This command will add the new user to the sudo group which grant them administrative privileges.

# Step 3: add a password for the new user.
```bash
passwd <newuser>
New password: new-password
Retype new password: new-password
```
replace <newuser> with your new user.
replace new-password with password of your choice.

# Step 4: Prevent the root user from connecting to the server via SSH.

Switch to the new user you just created:
```bash
sudo su -l <newuser>
```
Anytime that you are prompt to enter a password, enter the password you have just created.

open the ssh configuration file with vim.
```bash
sudo vim /etc/ssh/sshd_config
```
Inside the sshd_config file, look for "PermitRootLogin",
you can achieve this by scroll down or searching it with "/".

Once you find it, change the line to "PermitRootLogin no", 
then save and exit the file with ":wq".

now you need to copy the authorized_keys file to your user's ssh directory:
```bash
sudo cp /.ssh/authorized_keys /home/newuser/.ssh/
```

# step 5: Restart the SSH Service and try login SSH with your new user.
```bash
sudo systemctl restart ssh
```
Now that you have restarted SSH service, wait about 30 seconds for server reboot,
open a new terminal and try to login with the root user and make sure that you can't do so.

ssh -i root@server-ip

you should receive a similar error message like "Permission denied (publickey)."

Then try to login with the new user 
ssh -i newuser@server-ip

# Step 5: Install Nginx.
Now you are logged in with the new user, let's install Nginx service.
We will use apt to install the nginx package:
```bash
sudo apt update
```
you should always update the packages first to get the latest version, then:

```bash
sudo apt install nginx
```
Enter "Y" to continue installing the nginx package.

After you have installed nginx, you can check the status of nginx service:
```bash
sudo systemctl status nginx
```
# Step 6: Configure nginx to serve a sample website.
now that you have installed nginx, create a new directory like "my-site" in /var/www
```bash
cd /var/www
sudo mkdir my-site
```
now create an "index.html" document that will contain html code for a sample website, you are free to change the content inside, for example:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>
```
save the file and exit with :wq

# Step 7: Create the server block

```bash
cd /etc/nginx/sites-available
sudo vim my-site.conf

```
Create a new file in /etc/nginx/sites-available/ and paste the following nginx configuration code into it, you can make changes that serves your purposes.

```conf
server {
	listen 80
	
	root /var/www/my-site;
	
	index index.html index.htm;
	
	server_name _;
	
	location / {
		try_files $uri $uri/ =404;
	}
}
```
Change "my-site" to your the name of your folder if you chose a different name.

# Step 8: Create a symbolic link to your new config file in /etc/nginx/sites-enabled
```bash
sudo ln -s /etc/nginx/sites-available/my-site.conf /etc/nginx/sites-enabled/my-site conf
```
You will need to delete and unlink the default config file:
```bash
cd /etc/nginx/sites-available/
sudo unlink default
cd /etc/nginx/sites-enabled/
sudo rm default
```
Now test your nginx configuration:
```bash
sudo nginx -t
```

if you don't see error messages you can restart the nginx service:
```bash
sudo systemctl restart nginx
```
and run the curl command to get your sample HTML instead of the default nginx page:
```bash
curl your-ip-address
```
replace your-ip-address with the ip address of your ssh, you can find it with:
```bash
ip addr
```

Congratulation, you have succesfully created a new regular user and installed Nginx!
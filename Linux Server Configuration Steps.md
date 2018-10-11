# Linux Server Configuration Steps

## Installation process

### Be up to date with your packages

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Add User 

```bash
adduser grader
```

### Add grader authorized key

```bash
echo "<public_key>" >> /home/grader/.ssh/authorized_keys
```

### Add grader to sudoers list

```bash
sudo nano /etc/sudoers.d/grader-sudo
```
Add this line to the file
```bash
grader ALL=(ALL:ALL) NOPASSWD:ALL
```

### Pervent root login & password logins

```bash
sudo nano /etc/ssh/sshd_config
```
Change these values in the config
```
PermitRootLogin no
PasswordAuthentication no
UsePAM no
```

Restart ssh service
```bash
sudo systemctl restart ssh
```

### Change SSH port to 2200

```bash
sudo nano /etc/ssh/sshd_config
```

Add Following line

```
Port 2200
```

Restart ssh service

```bash
sudo systemctl restart sshd
```

### Setup firewall rules 

only allow connections from 2200, 80 and 123. **MAKE SURE YOU SETUP ALL THE RULES BEFORE ENABLING THE FIREWALL, OR YOU WILL GET LOCKED OUT FROM SSH OTHERWISE**

Deny all incoming connections
```bash
sudo ufw default deny incoming
```

Allow all outgoing 
```bash
sudo ufw default allow outgoing
```

Allow port 2200

```bash
sudo ufw allow 2200
```

Allow port 80

```bash
sudo ufw allow 80
```

Allow port 123

```bash
sudo ufw allow 123
```

Check the rules before enabling

```bash
sudo ufw show added
```

Enable the firewall

```bash
sudo ufw enable
```

Check the status of the firewall

```bash
sudo ufw status
```

### Install PostgreSQL

```bash
sudo apt-get update
sudo apt-get install postgresql
```

### Download and Setup Item Catalog application

```bash
git clone https://github.com/Scheercuzy/FSND-catalog-project.git
```

Download pip and install virtualenv

```bash
sudo apt-get install python3-pip
sudo pip install virtualenv
```

Follow the instructions on the Item Catalog App's README to install the rest

### Setup NGINX, uWSGI and systemd to serve flask app

Download Nginx

```bash
sudo apt-get install uwsgi uwsgi-plugin-python3
```

Create config file for your uwsgi app `/etc/uwsgi/apps-available/catalog-app.ini`

```
[uwsgi]
base = /path/to/project

master = true
processes = 2

plugins = python3
home = %(base)/path/to/venv
chdir = %(base)
module = catalog:app
env =

uid = grader
gui = users

socket = /tmp/catalog-app.sock
;chmod-socket = 664
vacuum = true

;socket = 0.0.0.0:5000
;protocol = http
```

Create symlink from `apps-available` to `apps-enabled`

```bash
sudo ln -s /etc/uwsgi/apps-available/catalog-app.ini /etc/uwsgi/apps-enabled/catalog-app.ini
```


Create service file for your uwsgi app in `/etc/systemd/system/catalog-app.uwsgi.service`

```
[Unit]
Description=uWSGI Catalog-App
After=syslog.target

[Service]
User=www-data
Group=www-data
ExecStart=/usr/bin/uwsgi --ini /etc/uwsgi/apps-enabled/catalog-app.ini
# Requires systemd version 211 or newer
RuntimeDirectory=uwsgi
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

Start service by running 

```bash
sudo systemctl start catalog-app.uwsgi.service
```

Enable it so it runs at boot

```bash
sudo systemctl enable catalog-app.uwsgi.service
```

Create Nginx config in `/etc/nginx/sites-available/catalog-app`

```
server {
    listen 80;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/tmp/catalog-app.sock;
    }
}
```

Create symlink from `sites-available` to `sites-enabled`

```bash
sudo ln -s /etc/nginx/sites-available/catalog-app /etc/nginx/sites-enabled
```

Restart nginx

```bash
sudo systemctl restart nginx
```


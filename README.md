# Django with Nginx and Gunicorn: A Self-Documentation Guide

This guide covers the setup of a Django web application using Nginx and Gunicorn. It includes steps for package updates, user creation, installation of dependencies, and configurations for Gunicorn and Nginx.

## Initial Setup and Package Updates

First, update your packages (optimal):

sudo apt update -y
sudo apt upgrade -y

## User Creation and Permissions

Create a non-root user (e.g., 'master'). Replace 'master' with your preferred username:

sudo adduser master
sudo usermod -aG sudo master
sudo usermod -aG www-data master

Log in with the new user:

su - master

## Installing Dependencies

Install necessary packages including PostgreSQL, Python, and Nginx (google it):

sudo apt install postgresql postgresql-contrib python3-pip python3-dev libpq-dev nginx -y

## Database Setup

Create a PostgreSQL database and user as needed.

## Python Virtual Environment

Create a new virtual environment. It's recommended to install this under your project directory. Add the virtual environment directory to .gitignore.

sudo -H pip3 install virtualenv

virtualenv venv

## Django Application Setup

Install Django web app dependencies:

pip install -r requirements.txt

# install gunicorn

pip install gunicorn

# If using PostgreSQL, install psycopg2:

pip install psycopg2-binary (check it)

# Add STATIC_ROOT to settings.py and update urls.py for static files.

# Migrate the database:

python manage.py makemigrations
python manage.py migrate

# Create a superuser and collect static files:

./manage.py createsuperuser
./manage.py collectstatic

## Gunicorn Service Configuration

# Create a Gunicorn service file(file name depends to you):

sudo nano /etc/systemd/system/gunicorn.service

# Add the following configuration to the .service file:

[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=username
Group=www-data
WorkingDirectory=/home/username/projectroot
ExecStart=/home/username/projectroot/env/bin/gunicorn --access-logfile - --workers 4 --bind unix:/home/username/projectroot/app.sock myproject.wsgi:application

[Install]
WantedBy=multi-user.target

# Reload and start the Gunicorn service:

sudo systemctl daemon-reload
sudo systemctl start gunicorn

## Nginx Configuration

# Create an Nginx configuration file:

sudo nano /etc/nginx/sites-available/myproject

# Add the following server block to the myproject configuration:

server {
    listen 80;
    server_name www.yourdomain.com;
    client_max_body_size 75M; 
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/username/projectroot;
    }
    location /media/ {
        root /home/username/projectroot;
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/username/projectroot/app.sock;
    }
}

server {
    listen 80;
    server_name nakeddomain.com;
    return 301 https://www.domain.com$request_uri;
}

# Link and enable the Nginx site:

sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

## Finalizing Setup

- Update DNS settings as needed. a point empty or @ to host ip, cname point www to naked domain name , optimal -  you can add aaaa ipv6 record.
- Check the status of Gunicorn and Nginx services.
- Start and enable the Gunicorn socket.
- Configure UFW to allow 'Nginx Full'.
- Install and configure HTTPS using Certbot.

## HTTPS Configuration

sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
sudo certbot renew --dry-run

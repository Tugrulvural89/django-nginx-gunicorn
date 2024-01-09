# django-nginx-gunicorn
Handle Django web app with nginx and gunicorn self documentation

# Before starting update packages it's optimal 
apt update -y
apt upgrade -y

# create user if you don't have any without root user. 'master' in this sample, change it with your prefered user name 

adduser master 

# assing user sudo command

usermod -aG sudo master

# add user to www-data. we will use this group in Gunicorn and nginx permissions who could override and write .conf and .service files

usermod -aG www-data master

# login with new user

su - master

# install dependencies packages. It can be changed Google it. 

sudo apt install postgresql postgresql-contrib python3-pip python3-dev libpq-dev nginx -y

# create postgresl database and user if you needed 

# create new virtualenv. You can change the name 'virtualenv'. I prefer install it under project file. don't forget to add .gitignore

sudo -H pip3 install virtualenv

# install Django web app dependencies with pip 

pip -r install requirements.txt

# check psycopg pip if you user postgress. Install it with binary 
pip install django gunicorn psycopg


# add static root to settings.py and statics to urls.py 

# migrate database

pyhon manage.py makemigrations
python manage.py migrate 

#create superuser and collecstatics

./manage.py createsuperuser
./manage.py collectstatic

# create gunicorn service file to start gunicorn process. File name optimal, you can change it. 

sudo nano /etc/systemd/system/gunicorn.service

# add below text to .service file

[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=username
Group=www-data
WorkingDirectory=/home/username/projectroot
ExecStart=/home/username/projectroot/projectenv/bin/gunicorn --access-logfile - --workers 4 --bind unix:/home/username/projectroot/name.sock djangoproject.wsgi:application

[Install]
WantedBy=multi-user.target

# update gunicorn service. Name will be .service file name 

sudo systemctl daemon-reload
sudo systemctl start name 

# create nginx .conf file 

sudo nano /etc/nginx/sites-available/djangoproject 

# add this line to djangoproject.conf 


server {
    listen 80;
    server_name www.domainname.com;
    client_max_body_size 75M;  
    location = /favicon.ico { access_log off; log_not_found off; }
    # Django media
    location /media/  {
        root /path/to/your/mysite/media; 
    }
    location /static/ {
        root /path/to/your/mysite/static; 
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/username/projectroot/name.sock;
    }
}

# if you haven't change dns settings yet. Update it as below text.

a point @ or empty to host ip
cname point www to host name 

# also if you user www subdomain you need to add www.domianname.com to servername 

# link to nginx available to enable folder

sudo ln -s /etc/nginx/sites-available/djangoproject /etc/nginx/sites-enabled/


# update and push nginx configurtion

sudo nginx -t

sudo systemctl restart nginx

# check status gunicorn and nginx. socket name can be diffrent. 

sudo systemctl status servicefilename

sudo systemctl status nginx

# start and enable gunicorn socket 

sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket

# check status 

sudo systemctl status gunicorn.socket

# enable uwf and port nginx

sudo ufw allow 'Nginx Full'

# install HTTPS 

sudo snap install --classic certbot 

sudo ln -s /snap/bin/certbot /usr/bin/certbot

sudo certbot --nginx

# test renewal

sudo certbot renew --dry-run








# Kittygram
***
## Kittygram is a project for photo sharing - https://kittygram.myvnc.com
***
## Technology:
* Python 3.10
* Django3
* Nginx
* Gunicorn
* React
* djangorestframework
* Certbot
***
## Installation on local server
1. Clone the project from the repository:
```git clone git@github.com:maxuver/infra_sprint1.git```
2. Set up a virtual environment: ```python3 -m venv venv```
3. Install dependencies: ```pip install -r requirements.txt```
4. Create a ***.env*** file in the root folder(in folder with manage.py), add SECRET_KEY
***
# Deploy the project to a remote server
***
 ### Connect to GitHub
Install Git: ```sudo apt install git```
Generate an SSH key pair: ```ssh-keygen```
Save the public key to a gitHub account: ```cat .ssh/id_rsa.pub```
Clone the project to a remote server: ```git clone git@github.com:Your_account/Project_name.git```  
***
### Start the backend
Install the package manager and the virtual environment utility on the server:  
```sudo apt install python3-pip python3-venv -y```  
Create a virtual environment:  
```python3 -m venv venv ```  
Activate virtual environment:  
```source venv/bin/activate```
Install dependencies:  
```pip install -r requirements.txt```  
Perform migrations:   
```python manage.py migrate```  
Create a superuser:  
```python manage.py createsuperuser```
In settings.py file in ALLOWED_HOSTS, add the IP address of the server, as well as '127.0.0.1', 'localhost' and the domain name.  
***
### Start the frontend
Install the npm package manager on the server:  
```curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\```  
```sudo apt-get install -y nodejs```
Install frontend dependencies from the ***infra_sprint1/frontend*** directory:  
``npm i``  
***
### Install and run Gunicorn
```pip install gunicorn==20.1.0```
Create a unit for gunicorn:  
```sudo nano /etc/systemd/system/gunicorn.service ```  
In the gunicorn.service file, describe the process configuration:
```html
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=yc-user 
WorkingDirectory=/home/yc-user/taski/backend/
ExecStart=/home/yc-user/taski/backend/venv/bin/gunicorn --bind 0.0.0.0.0:8000 backend.wsgi

[Install]
WantedBy=multi-user.target  
```
My example of gunicorn config [is here](https://github.com/maxuver/infra_sprint1/blob/main/infra/gunicorn_kittygram.service)

Start the gunicorn.service process:  
```sudo systemctl start gunicorn```  
Add the process to autostart:  
```sudo systemctl enable gunicorn```  
***
### Install Nginx
```sudo apt install nginx -y```  
Start Nginx:  
```sudo systemctl start nginx```  
Tell the firewall which ports should remain open:  
```sudo ufw allow 'Nginx Full'```  
```sudo ufw allow OpenSSH```  
Enable firewall:  
```sudo ufw enable```
***

### Build the frontend application statics
From the ***infra_sprint1/frontend*** directory, run:  
```npm run build```
```sudo cp -r /home/yc-user/infra_sprint1/frontend/build/. /var/www/kittygram/```  
Describe the settings for handling the frontend application static:   
``` sudo nano /etc/nginx/sites-enabled/default```  
Remove all settings from the file and add:  
```html
server {
    server_name YOUR_IP YOUR DOMAIN;
    client_max_body_size 32m;
    server_tokens off;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }
    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }
    location / {
        root /var/www/kittygram;
        index index.html index.html index.htm;
        try_files $uri /index.html;
    }
```
My example of nginx config [is here](https://github.com/maxuver/infra_sprint1/blob/main/infra/default)

***
### Describe the settings for working with the backend application
In the ***settings.py*** file, change(add):  
```STATIC_URL = 'static_backend'```
```STATIC_ROOT = BASE_DIR / 'static_backend'```  
With the virtual environment activated, run the command:  
```python manage.py collectstatic```
Go to the root of the project and run the command:  
```sudo cp -r infra_sprint1/backend/static_backend/ /var/www/kittygram/```  
***
### Configure encryption
Install ***certbot***:  
```sudo apt install snapd```

```sudo snap install core; sudo snap refresh core```  

```sudo snap install --classic certbot```

```sudo ln -s /snap/bin/certbot /usr/bin/certbot```

Let's run certbot and get the SSL certificate:  

```sudo certbot --nginx```

Reload the Nginx configuration:

```sudo systemctl reload nginx```  

Configure automatic SSL certificate renewal:  

```sudo certbot certificates```  

```sudo certbot renew --dry-run```  

### Setting up monitoring and collecting errors

Register on the site <small>[Uptimerobot](https://uptimerobot.com/).  
After that, on the main dashboard click the green button ***Add New Monitor***, in the opened window specify the monitor type: HTTP(s), come up with a name for this monitor, enter the URL you want to monitor.  


Author of the project | My website
------------- | -------------
[maxuver](https://github.com/maxuver) | <small>[maximpatsyuk.com](https://maximpatsyuk.com)

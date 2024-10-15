# Fastapi hosting using Gunicorn&Nginx

## Gunicorn

1. Create an account on any hosting platform and set up a new server on it.

2. Log in to the server through the terminal (on Linux and Mac) or the Command Prompt (on Windows)

```
    ssh username/root@server_ip
    password
```
3. Update the Virtual Machine

```
    sudo apt update
    sudo apt upgrade
```
4. Clone repository using personal access token from git

```
git clone -b branch_name https://github.com/repository
```
5. Install Gunicorn

```
sudo apt install gunicorn
```
6. Create virtual environment

```
    sudo apt install python3.11-venv
    python3 -m venv env
    source env/bin/activate
```
7. Enable Firewall
```
sudo ufw enable
```
- enable ports - example
```
    sudo ufw allow 8000
    sudo ufw allow 80
    sudo ufw allow 443
```
8. Install packages and test API

```
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

9. check on browser with IP and port number

- example
```
    http://0.0.0.0:8000/docs
```
10. Create fastapi-gunicorn.service

```
    sudo nano /etc/systemd/system/app_name.service
```
- copy the file with the following content
```
    [Unit]
    Description=Uvicorn service for FastAPI application

    [Service]
    ExecStart=/home/username/app_root/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
    WorkingDirectory=/home/username/app_root
    Restart=always
    User=username
    Group=username
    Environment="PATH=/home/username/app_root/venv/bin"

    [Install]
    WantedBy=multi-user.target
```

11. Enable Fastapi and start

```
    sudo systemctl enable app_name.service
    sudo systemctl start app_name.service
```

## Nginx configuration

1. Install Nginx

```
    sudo apt install nginx
````
2. Install certbort

```
    sudo apt install certbot python3-certbot-nginx -y
```
3. Install certification for domain name

```
    sudo certbot certonly --nginx -d domain_name
    sudo certbot certonly --nginx -d www.domain_name
```
4. connect domain

- create configuration file
```
    sudo nano /etc/nginx/sites-available/app_name
```
- and copy
```
    server {
        if ($host = www.domain_name) {
            return 301 https://$host$request_uri;
        } # managed by Certbot

        if ($host = domain_name) {
            return 301 https://$host$request_uri;
        } # managed by Certbot

        listen 80;
        server_name domain_name www.domain_name;

        # Redirect HTTP to HTTPS
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        client_max_body_size 5G;
        server_name domain_name www.domain_name;

        ssl_certificate /etc/letsencrypt/live/domain_name/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/domain_name/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        location / {
            proxy_pass http://127.0.0.1:port; # API address
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        # Additional configurations for your application can be added here
    }
```

5. Create symbolic link
```
    sudo ln -s /etc/nginx/sites-available/domain_name /etc/nginx/sites-enabled/
```
6. reload and start nginx

```
    sudo systemctl daemon-reload
    sudo systemctl start nginx
    or
    sudo systemctl restart nginx
```


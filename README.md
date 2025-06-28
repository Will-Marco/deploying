# Django Deployment with Gunicorn and Nginx

```bash
sudo apt install python3 python3-pip python3-venv python3-dev -y
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python3 manage.py migrate
python3 manage.py runserver

pip install gunicorn
sudo nano /etc/systemd/system/gunicorn.service
sudo systemctl daemon-reload
sudo systemctl enable gunicorn
sudo systemctl start gunicorn
sudo systemctl status gunicorn

sudo ufw allow 8000
sudo apt install nginx -y
rm -rf /etc/nginx/sites-available/default
rm -rf /etc/nginx/sites-enabled/default
sudo nano /etc/nginx/sites-available/default
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

/etc/systemd/system/gunicorn.service
[Unit]
Description=gunicorn daemon for Django project
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=path_file
ExecStart=path_file/venv/bin/gunicorn config.wsgi:application --bind 127.0.0.1:8000

[Install]
WantedBy=multi-user.target

/etc/nginx/sites-available/default
server {
    listen 80;
    server_name _;

    location = /favicon.ico { access_log off; log_not_found off; }

    location /static/ {
        alias path_file/static/;
    }

    location /media/ {
        alias path_file/media/;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}



# Django Project Setup and Deployment

This guide outlines the steps to set up and deploy a Django project using a Python virtual environment, Gunicorn, and Nginx on a Debian-based system.

## Prerequisites
- A Debian-based system (e.g., Ubuntu) with `sudo` privileges.
- Replace `path_file` in the configuration files with the absolute path to your Django project directory (e.g., `/home/user/project`).

## Setup Instructions

### 1. Install Dependencies
Install Python, pip, virtualenv, and development libraries:
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv python3-dev -y
```

### 2. Create and Activate Virtual Environment
Set up a Python virtual environment to isolate project dependencies:
```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Project Requirements
Install the required Python packages:
```bash
pip install -r requirements.txt
```

### 4. Apply Database Migrations
Run Django migrations to set up the database:
```bash
python3 manage.py migrate
```

### 5. Test the Development Server
Start the Django development server to verify the setup:
```bash
python3 manage.py runserver
```
Visit `http://localhost:8000` to confirm the application is running. Stop the server with `Ctrl+C` when done.

### 6. Install and Configure Gunicorn
Install Gunicorn to serve the Django application in production:
```bash
pip install gunicorn
```

Create a systemd service file for Gunicorn:
```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Add the following configuration, replacing `path_file` with your project directory:
```ini
[Unit]
Description=Gunicorn daemon for Django project
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=path_file
ExecStart=path_file/venv/bin/gunicorn config.wsgi:application --bind 127.0.0.1:8000

[Install]
WantedBy=multi-user.target
```

Enable and start the Gunicorn service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable gunicorn
sudo systemctl start gunicorn
sudo systemctl status gunicorn
```

### 7. Configure Firewall
Allow traffic on port 8000 (used by Gunicorn temporarily):
```bash
sudo ufw allow 8000
```

### 8. Install and Configure Nginx
Install Nginx to act as a reverse proxy:
```bash
sudo apt install nginx -y
```

Remove default Nginx configuration:
```bash
sudo rm -rf /etc/nginx/sites-available/default
sudo rm -rf /etc/nginx/sites-enabled/default
```

Create a new Nginx configuration:
```bash
sudo nano /etc/nginx/sites-available/default
```

Add the following configuration, replacing `path_file` with your project directory:
```nginx
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
```

Create a symbolic link to enable the configuration:
```bash
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
```

Test and restart Nginx:
```bash
sudo nginx -t
sudo systemctl restart nginx
```

## Notes
- Ensure `path_file/static/` and `path_file/media/` directories exist and are populated with static and media files. Run `python3 manage.py collectstatic` to collect static files.
- For production, secure your server with HTTPS by obtaining an SSL certificate (e.g., using Let's Encrypt).
- Adjust the `User` in the Gunicorn service file to a non-root user for better security.
- If you encounter issues, check logs:
  - Gunicorn: `sudo journalctl -u gunicorn`
  - Nginx: `sudo tail -f /var/log/nginx/error.log`
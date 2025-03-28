# Deploying a Django Application to AWS EC2

## Prerequisites
- An AWS account
- A Django application ready for deployment
- Basic familiarity with Linux command line
- SSH client (PuTTY for Windows or Terminal for Mac/Linux)

## Step 1: Create an EC2 Instance
1. Log in to AWS Management Console
2. Navigate to EC2 Dashboard
3. Click "Launch Instance"
4. Choose an Amazon Machine Image (AMI)
   - **Recommended**: Ubuntu Server LTS
5. Select instance type
   - `t2.micro` is free tier eligible
6. Configure security group:
   - Allow SSH (Port 22)
   - Allow HTTP (Port 80)
   - Allow HTTPS (Port 443)
   - Allow custom TCP for your Django app (e.g., Port 8000)

## Step 2: Set Up SSH Access
1. Create a new key pair or use existing
2. Download the `.pem` private key file
3. Set appropriate permissions:
   ```bash
   chmod 400 your-key.pem
   ```
4. Connect to your instance:
   ```bash
   ssh -i your-key.pem ubuntu@your-ec2-public-dns
   ```

## Step 3: Prepare the Server
```bash
# Update system packages
sudo apt-get update
sudo apt-get upgrade -y

# Install Python and essential tools
sudo apt-get install -y python3-pip python3-dev nginx git

# Install virtual environment
sudo pip3 install virtualenv
```

## Step 4: Set Up Project Environment
```bash
# Clone your project or transfer files
git clone your-project-repository.git
cd your-project-directory

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install project dependencies
pip install -r requirements.txt
pip install gunicorn
```

## Step 5: Configure Database
- If using PostgreSQL:
```bash
sudo apt-get install -y postgresql postgresql-contrib python3-psycopg2
sudo -u postgres psql
```
Create database and user for your Django project

## Step 6: Configure Django Settings
1. Update `settings.py`:
   - Set `DEBUG = False`
   - Add your EC2 public DNS to `ALLOWED_HOSTS`
   - Configure database settings
   - Set secure secret key

2. Run migrations:
```bash
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic
```

## Step 7: Set Up Gunicorn
1. Create Gunicorn systemd service file:
```bash
sudo nano /etc/systemd/system/gunicorn.service
```

2. Add configuration:
```ini
[Unit]
Description=Gunicorn daemon
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/path/to/your/project
ExecStart=/path/to/your/venv/bin/gunicorn \
    --workers 3 \
    --bind 127.0.0.1:8000 \
    your_project.wsgi:application

[Install]
WantedBy=multi-user.target
```

3. Start and enable Gunicorn:
```bash
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

## Step 8: Configure Nginx
1. Create Nginx configuration:
```bash
sudo nano /etc/nginx/sites-available/your_project
```

2. Add server block:
```nginx
server {
    listen 80;
    server_name your-domain-or-ip;

    location = /favicon.ico { access_log off; log_not_found off; }
    
    location /static/ {
        root /path/to/your/project;
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

3. Enable site and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/your_project /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx

sudo systemctl daemon-reload 
sudo systemctl restart gunicorn
sudo systemctl restart nginx 
```

## Step 9: Set Up SSL (Optional)
- Use Let's Encrypt with Certbot for free SSL
```bash
sudo apt-get install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

## Troubleshooting
- Check Gunicorn logs: 
  ```bash
  sudo journalctl -u gunicorn
  ```
- Check Nginx logs: 
  ```bash
  sudo tail /var/log/nginx/error.log
  ```
- Ensure all file permissions are correct
- Verify firewall and security group settings

## Maintenance
- Regular updates: 
  ```bash
  sudo apt-get update && sudo apt-get upgrade
  ```
- Periodic security checks
- Monitor server performance

## Common Pitfalls
- Always double-check file paths in configuration files
- Ensure your `settings.py` is configured for production
- Make sure all required environment variables are set
- Verify database connection settings

## Notes
- Replace placeholders like `your-project-repository.git`, `/path/to/your/project`, etc., with your actual project details
- The guide assumes an Ubuntu-based EC2 instance
- Adjust commands for other Linux distributions as needed
```

- Additional notes and troubleshooting tips
- Consistent formatting

You can directly copy this into your GitHub repository's README.md file. Would you like me to make any further modifications?

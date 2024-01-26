# Healthchecks.io: Cron Job Monitoring () Installation Guide 
### Prerequisites
- A Linux server (e.g., Ubuntu 20.04)
- Basic understanding of the Linux command line
- Sudo or root access to the server

### Step 1: Install Required Packages
1. Update package lists:
   ```bash
   sudo apt update
   ```
2. Install Python, pip, and other essentials:
   ```bash
   sudo apt install python3 python3-pip python3-venv git
   ```

### Step 2: Clone Healthchecks Repository
1. Clone the Healthchecks repository:
   ```bash
   git clone https://github.com/healthchecks/healthchecks.git
   cd healthchecks
   ```

### Step 3: Set Up Python Virtual Environment
1. Create a virtual environment:
   ```bash
   python3 -m venv venv
   ```
2. Activate the virtual environment:
   ```bash
   source venv/bin/activate
   ```

### Step 4: Install Python Dependencies
1. Install requirements using pip:
   ```bash
   pip install -r requirements.txt
   ```

### Step 5: Configure Healthchecks
1. Copy the local settings template:
   ```bash
   cp hc/local_settings.py.example hc/local_settings.py
   ```
2. Edit `hc/local_settings.py` to suit your environment (use a text editor like nano or vim).

   2. Update the following settings in the file:

   ```python
   # Admins who receive error notifications
   ADMINS = [("Admin", "your@email.com")]

   # Host/domain names that your site can serve
   ALLOWED_HOSTS = ["yourdomain.com", "localhost", "*"]

   # Database configuration
   DATABASES = {
       "default": {
           "ENGINE": "django.db.backends.postgresql", # Leave as per your need sql or postgresql
           "NAME": "db_name",
           "USER": "db_user",
           "PASSWORD": "db@password",
           "HOST": "",  # Leave blank unless your database is on a separate server
           "PORT": "",  # Leave blank to use the default port
       }
   }

   # Debug mode
   DEBUG = False  # Set to False in production

   # Email configuration
   EMAIL_BACKEND = "django.core.mail.backends.smtp.EmailBackend"
   EMAIL_HOST = "smtp.yourdomain.com"
   EMAIL_PORT = 465
   EMAIL_HOST_USER = "apikey"
   EMAIL_HOST_PASSWORD = "YOUR_API_KEY"
   EMAIL_USE_TLS = True
   DEFAULT_FROM_EMAIL = "support@yourdomain.com"
   SERVER_EMAIL = "support@yourdomain.com"

   # Site settings
   SITE_NAME = " HealthChecks Server"
   SITE_ROOT = "https://Your Domain"

   # Secret key for cryptographic signing
   SECRET_KEY = "d45hiybqc9toeok-e2w_!wftsjf%&ok"   # Generate any random key

   # Other settings
   REGISTRATION_OPEN = False
   SHELL_ENABLED = True
   ```


### Step 6: Initialize Database
1. Apply migrations:
   ```bash
   ./manage.py migrate
   ```
2. Create an admin user (follow prompts):
   ```bash
   ./manage.py createsuperuser
   ```

### Step 7: Collect Static Files
1. Collect static files:
   ```bash
   ./manage.py collectstatic
   ```

### Step 8: Install and Configure Gunicorn
1. Install Gunicorn:
   ```bash
   pip install gunicorn
   ```
2. Test Gunicorn's ability to serve the project:
   ```bash
   gunicorn hc.wsgi:application --bind 0.0.0.0:8000
   ```
3. Visit `http://YOUR_SERVER_IP:8000` to see if the app is running.

### Step 9: Set Up Gunicorn as a System Service
1. Create a systemd service file:
   ```bash
   sudo nano /etc/systemd/system/gunicorn.service
   ```
2. Add the following content to the file, adjust paths accordingly:
   ```ini
   [Unit]
   Description=gunicorn daemon
   After=network.target

   [Service]
   User=YOUR_USERNAME
   Group=www-data
   WorkingDirectory=/path/to/healthchecks
   ExecStart=/path/to/healthchecks/venv/bin/gunicorn --workers 3 --bind unix:/path/to/healthchecks/healthchecks.sock hc.wsgi:application

   [Install]
   WantedBy=multi-user.target
   ```
3. Start and enable the service:
   ```bash
   sudo systemctl start gunicorn
   sudo systemctl enable gunicorn
   ```
### Optional SSL Installation 
### Installing OpenSSL and Generating SSL Certificates
For SSL installation, we'll use OpenSSL to create a self-signed certificate. Note that self-signed certificates are fine for testing but are not recommended for production use due to security concerns. For a production environment, consider using a certificate issued by a trusted certificate authority (CA).

1. **Install OpenSSL:**
   ```bash
   sudo apt install openssl
   ```

2. **Generate a Self-Signed SSL Certificate:**
   ```bash
   sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/healthchecks-selfsigned.key -out /etc/ssl/certs/healthchecks-selfsigned.crt
   ```
   Follow the prompts to complete the certificate creation.

3. **Update Nginx to Use SSL:**
   Modify your Nginx configuration to use the SSL certificate. In the server block for port 443, add:
   ```nginx
   ssl_certificate /etc/ssl/certs/healthchecks-selfsigned.crt;
   ssl_certificate_key /etc/ssl/private/healthchecks-selfsigned.key;
   ```

4. **Restart Nginx:**
   ```bash
   sudo systemctl restart nginx
   ```
   
### Step 10: Install and Configure Nginx
1. Install Nginx:
   ```bash
   sudo apt install nginx
   ```
2. Create a new Nginx server block:
   ```bash
   sudo nano /etc/nginx/sites-available/healthchecks
   ```
3. Add the following configuration, adjust paths and domain names:
   ```nginx
   server {
       listen 80;
       server_name YOUR_DOMAIN.com;

       location = /favicon.ico { access_log off; log_not_found off; }
       location /static/ {
           alias /path/to/healthchecks/static-collected/;
       }

       location / {
           include proxy_params;
           proxy_pass http://unix:/path/to/healthchecks/healthchecks.sock;
       }
   }
   ```
4. Enable the new configuration:
   ```bash
   sudo ln -s /etc/nginx/sites-available/healthchecks /etc/nginx/sites-enabled
   sudo nginx -t
   sudo systemctl restart nginx
   ```

### Step 11: Final Checks
1. Visit `http://YOUR_DOMAIN.com` in your browser. The Healthchecks application should now be live.
2. Make sure to open the necessary ports (e.g., 80, 443) in your server's firewall.

### Additional Notes
- **Security**: Consider setting up a firewall (e.g., using `ufw`) and Fail2Ban.
- **SSL**: To add HTTPS, use Let's Encrypt/Certbot.
- **Backups**: Regularly back up your database and static files.

This guide provides an overview. Depending on your server's configuration and specific requirements, you may need to adjust some steps.

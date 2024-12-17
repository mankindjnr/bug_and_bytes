To deploy your containerized Django application and serve it securely via amos.chamazetu.com using Docker, Nginx, and Let's Encrypt for HTTPS, follow this step-by-step guide:

1. Prerequisites
   You should already have a VM (Virtual Machine) set up.
   Docker and Docker Compose should be installed on your VM.
   Nginx should be installed.
   DNS for amos.chamazetu.com should point to your VM's public IP address.
2. Run Your Docker Container
   Assuming you've already pulled the Docker image from the GitHub container registry, you can run the container with:

```bash
docker run -d --name amos-portfolio-app -p 8000:8000 your_image_name
Replace your_image_name with the actual name of your Docker image.

-d runs the container in the background.
--name amos-portfolio-app assigns a name to the container.
-p 8000:8000 maps port 8000 of the host to port 8000 of the container.
```

3. Install and Configure Nginx
   Install Nginx:

If Nginx isn't installed, install it using:

```bash
sudo apt update
sudo apt install nginx
Create an Nginx Configuration for amos.chamazetu.com:

Create a new Nginx configuration file for your site:
```

```bash
sudo nano /etc/nginx/sites-available/amos.chamazetu.com
```

Add the following configuration:

```conf
server {
    listen 80;
    server_name amos.chamazetu.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

This configuration listens for HTTP traffic on port 80 and forwards it to the Docker container running on port 8000.
server_name amos.chamazetu.com; specifies the domain name.
Enable the Nginx Configuration:

Create a symbolic link to enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/amos.chamazetu.com /etc/nginx/sites-enabled/
```

Test and Reload Nginx:
Test the configuration for syntax errors:

```bash
sudo nginx -t
```

If the test is successful, reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

4. Allow HTTP and HTTPS Traffic on the Firewall
   If you're using UFW (Uncomplicated Firewall), allow Nginx traffic:

```bash
sudo ufw allow 'Nginx Full'
```

5. Obtain SSL Certificate Using Let's Encrypt
   Let's Encrypt provides free SSL certificates. Use Certbot to obtain a certificate and configure Nginx.

Install Certbot:
Install Certbot and the Nginx plugin:

```bash
sudo apt install certbot python3-certbot-nginx
```

Obtain and Install the SSL Certificate:
Run Certbot with the Nginx plugin to automatically obtain and install the certificate:

```bash
sudo certbot --nginx -d amos.chamazetu.com
```

Certbot will prompt you for your email address and ask you to agree to the terms of service.
Certbot will automatically edit your Nginx configuration to use SSL.
Verify SSL Installation:

Once the process is complete, Certbot will configure Nginx to use the newly obtained certificate. Verify the installation by visiting https://amos.chamazetu.com in your web browser. You should see the HTTPS version of your site.

6. Auto-Renewal of SSL Certificates
   Certbot sets up a cron job for automatic renewal. You can test the renewal process with:

```bash
sudo certbot renew --dry-run
```

This ensures your SSL certificate will renew automatically without any issues.

7. Check Logs and Monitor
   Monitor Nginx and Docker logs to ensure everything is working correctly:

Nginx logs: Check for any errors or issues.

```bash
sudo tail -f /var/log/nginx/error.log
```

Docker container logs: View logs for the running application.

```bash
docker logs amos-portfolio-app
```

Summary
Run the Docker container using docker run.
Install and configure Nginx to serve as a reverse proxy.
Allow HTTP/HTTPS traffic using the firewall.
Use Certbot to obtain and configure SSL certificates with Let's Encrypt.
Test and verify that amos.chamazetu.com is accessible over HTTPS.
Ensure auto-renewal of SSL certificates.
Monitor Nginx and Docker logs for any issues.
This setup provides a secure, containerized deployment of your Django application accessible at amos.chamazetu.com.

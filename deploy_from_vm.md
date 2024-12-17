## Deploy webapp from a GCP VM instance

This notebook will guide you through the process of deploying a webapp from a GCP VM instance. The webapp will be a multi-container applicatio that consists of a frotend, backend, redis, nginx, and celery workers.

- To access your services over the internet, you will need to ensure that:

1. Firewall Rules: The appropriate firewall rules are configured to allow traffic on the ports your services are running on.
2. External IP: Your VM instance has an external IP address.
3. Domain Name: You have a domain name that you can use to access your services.
4. Port Mapping: Your services are correctly mapped to the VM's external IP address.

- To configure firewall rules:
  By default, GCP blocks most incoming traffic. You need to create firewall rules to allow traffic on the required ports.

**Create Firewall Rules via Console:**

- Go to the VPC network -> Firewall rules in the GCP Console.
- Click Create firewall rule.
- Configure the rule:
- Name: A descriptive name, e.g., allow-8000-9400.
- Targets: Select the targets (usually All instances in the network or specific instances).
- Source IP ranges: Usually 0.0.0.0/0 to allow traffic from any IP.
- Protocols and ports: Specify the protocols and ports to allow, e.g., tcp:8000,9400.

**Access Your Services**
After setting up the firewall rules, you can access your services using the external IP address of your VM and the respective ports. For example, if your VM’s external IP is 34.123.45.67:

Backend service on port 9000: http://34.123.45.67:9000
Frontend service on port 8400: http://34.123.45.67:8400

## Setup webapplication with a domain name from Namecheap

**and run it on https with nginx as a reverse proxy**

1. Update Docker COmpose File

- Add an Nginx service to your docker-compose file.
- Configure nginx to serve as a reverse proxy for your frontend and backend services.
- update the frontend and backend services to remove ssl hadling since Nginx will manage SSL.(if you have ssl handling in your services, i.e self-assigned certificates)

2. Configure DNS

   - Update the DNS records for your domain to point to the external IP address of your VM instance.
   - Add an A record for the domain name and point it to the external IP address of your VM instance.

3. Configure Nginx

   - Configure Nginx to handle SSL using Let's Encrypt.
   - Set up Nginx configuration files for your frontend and backend services.

4. Generate SSL Certificates
   - Use Certbot to generate SSL certificates for your domain.
   - Certbot will automatically configure Nginx to use the SSL certificates.

## STEP BY STEP GUIDE

***
shift the assignments to this
```yaml
chamazetu_database:
    image: postgres:16
    container_name: chamazetu_database
    restart: always
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_HOST_AUTH_METHOD: scram-sha-256
      POSTGRES_INITDB_ARGS: "--data-checksums"
    volumes:
      - chamazetu_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - chamazetu_network
```

```yaml
services:
  message_broker:
    image: redis:latest
    ports:
      - "6380:6379"
    networks:
      - chamazetu_network

  pgbouncer:
    image: edoburu/pgbouncer:latest
    restart: always
    environment:
      - DB_USER=${SUPABASE_DB_USER}
      - DB_PASSWORD=${SUPABASE_DB_PASSWORD}
      - DB_HOST=${SUPABASE_DB_HOST}
      - AUTH_TYPE=scram-sha-256
      - POOL_MODE=transaction
      - LISTEN_PORT=6432
    ports:
      - "6432:6432"
    volumes:
      - ./pgbouncer.ini:/etc/pgbouncer/pgbouncer.ini
      - ./userlist.txt:/etc/pgbouncer/userlist.txt
    networks:
      - chamazetu_network

  chamazetu_frontend:
    image: ghcr.io/mankindjnr/chamazetu_frontend:latest
    command: python manage.py runserver 0.0.0.0:8000
    environment:
      - TZ=Africa/Nairobi
      - DATABASE_URL=postgres://${SUPA_DB_USER}:${SUPA_DB_PASSWORD}@${SUPA_DB_HOST}:5432/${SUPA_DB_NAME}
    depends_on:
      - message_broker
    networks:
      - chamazetu_network
    deploy:
      replicas: 2

  chamazetu_backend:
    image: ghcr.io/mankindjnr/chamazetu_backend:latest
    command: uvicorn app.main:app --host 0.0.0.0 --port 9400 --reload
    environment:
      - TZ=Africa/Nairobi
      - DATABASE_URL=postgres://${SUPABASE_DB_USER}:${SUPABASE_DB_PASSWORD}@pgbouncer:6432/${SUPABASE_DB_NAME}
    depends_on:
      - pgbouncer
    networks:
      - chamazetu_network
    deploy:
      replicas: 4

  celery_worker:
    image: ghcr.io/mankindjnr/celery_worker:latest
    command: celery -A frontend_chamazetu.celery worker -l INFO
    environment:
      - TZ=Africa/Nairobi
      - DATABASE_URL=postgres://${SUPA_DB_USER}:${SUPA_DB_PASSWORD}@${SUPA_DB_HOST}:5432/${SUPA_DB_NAME}
      - C_FORCE_ROOT=true
    depends_on:
      - message_broker
    networks:
      - chamazetu_network
    deploy:
      replicas: 3

  celery_beat_scheduler:
    image: ghcr.io/mankindjnr/celery_beat_scheduler:latest
    command: celery -A frontend_chamazetu.celery beat -l INFO
    environment:
      - TZ=Africa/Nairobi
      - DATABASE_URL=postgres://${SUPA_DB_USER}:${SUPA_DB_PASSWORD}@${SUPA_DB_HOST}:5432/${SUPA_DB_NAME}
    depends_on:
      - message_broker
    networks:
      - chamazetu_network

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - chamazetu_frontend
      - chamazetu_backend
    networks:
      - chamazetu_network

networks:
  chamazetu_network:
    driver: bridge
```

### Configure DNS

- Log in to your Namecheap account and set the A record for your domain to point to the external IP address of your GCP VM.

### Configure Nginx

Before running the Certbot command, the web application (specifically the Nginx server) should be running. This is because Certbot needs to place a challenge file in the .well-known/acme-challenge/ directory and have it served over HTTP to verify domain ownership.

Here's a detailed step-by-step process to ensure everything is set up correctly:

Step-by-Step Process
Step 1: Set Up DNS Records
Ensure your DNS records are correctly set up in Namecheap:

A Record for Root Domain (chamazetu.com):

Host: @
Value: 35.239.35.128 (your GCP VM's external IP address)
A Record for www Subdomain (www.chamazetu.com):

Host: www
Value: 35.239.35.128 (your GCP VM's external IP address)
Step 2: Update Nginx Configuration
Update your nginx.conf to handle the ACME challenges and serve your application:

```conf
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    upstream chamazetu_frontend {
        server chamazetu_frontend:8000;
        server chamazetu_frontend:8001;
    }

    upstream chamazetu_backend {
        server chamazetu_backend:9400;
        server chamazetu_backend:9401;
        server chamazetu_backend:9402;
        server chamazetu_backend:9403;
    }

    sendfile on;
    keepalive_timeout 65;

    server {
        listen 80;
        server_name chamazetu.com www.chamazetu.com;

        location /.well-known/acme-challenge/ {
            root /var/www/certbot;
        }

        location / {
            return 301 https://$host$request_uri;
        }
    }
}
```

Navigate to the directory containing your docker-compose.yml file and create the required directories:

```bash
cd /path/to/your/deployment_chamazetu
mkdir -p certbot/conf certbot/www
```

Step 3: Ensure Nginx and Other Services are Running
Start your services using Docker Compose:

```bash
docker-compose up -d
#Make sure Nginx is running and serving your application:


docker-compose ps
#You should see your Nginx container along with other services up and running.

Step 4: Run Certbot
#Now, with Nginx running, you can run the Certbot command to obtain SSL certificates:

docker run -it --rm --name certbot \
    -v "$(pwd)/certbot/conf:/etc/letsencrypt" \
    -v "$(pwd)/certbot/www:/var/www/certbot" \
    certbot/certbot certonly --webroot \
    --webroot-path=/var/www/certbot \
    --email njorogekairu@chamazetu.com \
    --agree-tos \
    --no-eff-email \
    -d chamazetu.com \
    -d www.chamazetu.com -v
```

after running the certbot command, you should see a success message indicating that the SSL certificates have been successfully obtained. now update the nginx.conf file to include the SSL configuration:

```conf
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    upstream chamazetu_frontend {
        server chamazetu_frontend:8000;
        server chamazetu_frontend:8001;
    }

    upstream chamazetu_backend {
        server chamazetu_backend:9400;
        server chamazetu_backend:9401;
        server chamazetu_backend:9402;
        server chamazetu_backend:9403;
    }

    sendfile on;
    keepalive_timeout 65;

    server {
        listen 80;
        server_name chamazetu.com www.chamazetu.com;

        location /.well-known/acme-challenge/ {
            root /var/www/certbot;
        }

        location / {
            return 301 https://$host$request_uri;
        }
    }

    server {
        listen 443 ssl;
        server_name chamazetu.com www.chamazetu.com;

        ssl_certificate /etc/letsencrypt/live/chamazetu.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/chamazetu.com/privkey.pem;

        location / {
            proxy_pass http://chamazetu_frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /api {
            proxy_pass http://chamazetu_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Summary

- Set DNS records for both chamazetu.com and www.chamazetu.com.
- Update Nginx configuration to serve ACME challenge files.
- Start Nginx and other services using Docker Compose.
- Run Certbot to request SSL certificates.
- By ensuring that your web application, especially Nginx, is running before executing the Certbot command, Certbot can successfully place and serve the challenge files needed for domain verification.

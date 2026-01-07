# Multi-Service Deployment Guide

## Overview
This guide covers deploying a full-stack application with three services:
- **Backend API** (Spring Boot on port 7000)
- **Frontend User Panel** (Next.js on port 3000)
- **Admin Panel** (Next.js on port 4000)

All services run in Docker containers behind an Nginx reverse proxy with SSL/TLS certificates.

---

## Architecture

```
Internet → Nginx (SSL Termination)
            ├── consultinghub.xyz → Frontend (User Panel)
            ├── admin.consultinghub.xyz → Admin Panel
            └── api.consultinghub.xyz → Backend API
```

---

## Step 1: DNS Configuration

Configure A records in your DNS provider (e.g., Namecheap):

| Type | Host  | Value          | TTL       |
|------|-------|----------------|-----------|
| A    | @     | [SERVER_IP]    | Automatic |
| A    | www   | [SERVER_IP]    | Automatic |
| A    | admin | [SERVER_IP]    | Automatic |
| A    | api   | [SERVER_IP]    | Automatic |

---

## Step 2: Server Setup

### Create Project Directory
```bash
mkdir nexadocs
cd nexadocs
```

### Clone Repositories
```bash
# Backend
git clone https://[TOKEN]@github.com/[ORG]/NexaDoc.git

# Frontend
git clone https://[TOKEN]@github.com/[ORG]/NexaDoc_front.git

# Admin
git clone https://[TOKEN]@github.com/[ORG]/NexaDoc_Admin.git
```

### Checkout Development Branch
```bash
cd NexaDoc && git checkout develop && git pull && cd ..
cd NexaDoc_front && git checkout develop && git pull && cd ..
cd NexaDoc_Admin && git checkout develop && git pull && cd ..
```

---

## Step 3: Docker Compose Configuration

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  # Backend API Service
  backend:
    image: nexa_doc_back_image
    build:
      context: ./NexaDoc
      dockerfile: Dockerfile
    container_name: nexa_doc_back_container
    env_file:
      - ./NexaDoc/.env
    volumes:
      - ./NexaDoc/firebase-service-account.json:/app/firebase-service-account.json
    environment:
      - SERVER_FORWARD_HEADERS_STRATEGY=framework
      - SERVER_TOMCAT_REMOTEIP_PROTOCOL_HEADER=x-forwarded-proto
      - SERVER_TOMCAT_REMOTEIP_REMOTE_IP_HEADER=x-forwarded-for
    restart: always
    expose:
      - "7000"
    networks:
      - nexadocs_network

  # Frontend User Panel
  frontend:
    image: nexadoc_front-nextjs
    build:
      context: ./NexaDoc_front
      args:
        NEXT_PUBLIC_BASE_URL: ${NEXT_PUBLIC_BASE_URL}
        NEXT_PUBLIC_SITE_URL: ${NEXT_PUBLIC_SITE_URL}
        NEXT_PUBLIC_FIREBASE_API_KEY: ${NEXT_PUBLIC_FIREBASE_API_KEY}
        NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN: ${NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN}
        NEXT_PUBLIC_FIREBASE_PROJECT_ID: ${NEXT_PUBLIC_FIREBASE_PROJECT_ID}
        NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET: ${NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET}
        NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID: ${NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID}
        NEXT_PUBLIC_FIREBASE_APP_ID: ${NEXT_PUBLIC_FIREBASE_APP_ID}
        NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID: ${NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID}
        NEXT_PUBLIC_FIREBASE_VAPID_KEY: ${NEXT_PUBLIC_FIREBASE_VAPID_KEY}
      dockerfile: Dockerfile
    container_name: nexadocs-user-panel
    expose:
      - "3000"
    env_file:
      - .env
    environment:
      - NODE_ENV=${NODE_ENV}
    restart: unless-stopped
    networks:
      - nexadocs_network

  # Admin Panel
  admin:
    image: nexadoc_admin-nextjs
    build:
      context: ./NexaDoc_Admin
      args:
        NEXT_PUBLIC_BASE_URL: ${NEXT_PUBLIC_BASE_URL}
        NEXT_PUBLIC_SITE_URL: ${NEXT_PUBLIC_SITE_URL}
        NEXT_PUBLIC_FIREBASE_API_KEY: ${NEXT_PUBLIC_FIREBASE_API_KEY}
        NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN: ${NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN}
        NEXT_PUBLIC_FIREBASE_PROJECT_ID: ${NEXT_PUBLIC_FIREBASE_PROJECT_ID}
        NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET: ${NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET}
        NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID: ${NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID}
        NEXT_PUBLIC_FIREBASE_APP_ID: ${NEXT_PUBLIC_FIREBASE_APP_ID}
        NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID: ${NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID}
        NEXT_PUBLIC_FIREBASE_VAPID_KEY: ${NEXT_PUBLIC_FIREBASE_VAPID_KEY}
      dockerfile: Dockerfile
    container_name: nexadocs-admin-panel
    expose:
      - "4000"
    env_file:
      - .env
    environment:
      - NODE_ENV=${NODE_ENV}
      - NEXT_PUBLIC_SITE_URL=https://admin.consultinghub.xyz
    restart: unless-stopped
    networks:
      - nexadocs_network

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./certbot/www:/var/www/certbot
      - ./certbot/conf:/etc/letsencrypt
    depends_on:
      - backend
      - frontend
      - admin
    networks:
      - nexadocs_network
    restart: unless-stopped

networks:
  nexadocs_network:
    driver: bridge
```

---

## Step 4: Environment Variables

### Backend Configuration (`NexaDoc/.env`)
```bash
# Database
DB_HOST=[DB_HOST]
DB_PORT=[DB_PORT]
DB_NAME=[DB_NAME]
DB_USER=[DB_USER]
DB_PASSWORD=[DB_PASSWORD]
DB_SCHEMA=[DB_SCHEMA]

# File Upload
AVATAR_MAX_SIZE=5242880
AVATAR_ALLOWED_FORMATS=image/jpeg,image/png

# Firebase
USER_JOIN_DATE_FORMAT=yyyy-MM-dd
GOOGLE_APPLICATION_CREDENTIALS=/app/firebase-service-account.json
BUCKET_NAME=[BUCKET_NAME]
FIREBASE_URL=https://firebasestorage.googleapis.com/v0/b/

# SMS Service
SMS_LOGIN_API_URL=[SMS_API_URL]
SMS_SEND_API_URL=[SMS_SEND_URL]

# Payment Gateway
SSL_COMMERCE_STORE_ID=[STORE_ID]
SSL_COMMERCE_STORE_PASSWORD=[STORE_PASSWORD]

# Email Service
RESEND_API_KEY=[RESEND_KEY]
RESEND_API_URL=https://api.resend.com/emails
RESEND_FROM_EMAIL=[FROM_EMAIL]

# Zoom Integration
ZOOM_ACCOUNT_ID=[ZOOM_ACCOUNT_ID]
ZOOM_CLIENT_ID=[ZOOM_CLIENT_ID]
ZOOM_CLIENT_SECRET=[ZOOM_CLIENT_SECRET]
ZOOM_API_BASE_URL=https://api.zoom.us/v2
```

### Frontend Configuration (`nexadocs/.env`)
```bash
# Site URLs
NEXT_PUBLIC_SITE_URL=https://consultinghub.xyz/
NEXT_PUBLIC_BASE_URL=https://api.consultinghub.xyz/api/v1

# Firebase Configuration
NEXT_PUBLIC_FIREBASE_API_KEY=[FIREBASE_API_KEY]
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=[FIREBASE_AUTH_DOMAIN]
NEXT_PUBLIC_FIREBASE_PROJECT_ID=[FIREBASE_PROJECT_ID]
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=[FIREBASE_STORAGE_BUCKET]
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=[FIREBASE_SENDER_ID]
NEXT_PUBLIC_FIREBASE_APP_ID=[FIREBASE_APP_ID]
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=[FIREBASE_MEASUREMENT_ID]
NEXT_PUBLIC_FIREBASE_VAPID_KEY=[FIREBASE_VAPID_KEY]

# Environment
NODE_ENV=production
```

### Firebase Service Account (`NexaDoc/firebase-service-account.json`)
```json
{
  "type": "service_account",
  "project_id": "[PROJECT_ID]",
  "private_key_id": "[KEY_ID]",
  "private_key": "[PRIVATE_KEY]",
  "client_email": "[CLIENT_EMAIL]",
  "client_id": "[CLIENT_ID]",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "[CERT_URL]",
  "universe_domain": "googleapis.com"
}
```

---

## Step 5: SSL Setup Script

Create `multi-service-ssl-setup.sh`:

```bash
#!/bin/bash

echo "Setting up SSL for all consultinghub.xyz services..."

# Stop all running services
echo "Stopping all services..."
docker-compose down 2>/dev/null || true
docker stop nginx-proxy temp-nginx 2>/dev/null || true
docker rm nginx-proxy temp-nginx 2>/dev/null || true

# Create directories
echo "Creating directories..."
mkdir -p nginx certbot/www certbot/conf

# Create temporary nginx for certificate generation
cat > nginx/nginx.conf << 'EOF'
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name consultinghub.xyz www.consultinghub.xyz admin.consultinghub.xyz api.consultinghub.xyz;

        location /.well-known/acme-challenge/ {
            root /var/www/certbot;
        }

        location / {
            root /var/www/html;
            index index.html;
        }
    }
}
EOF

# Start temporary nginx
echo "Starting temporary nginx..."
docker run -d --name temp-nginx \
  -p 80:80 \
  -v $(pwd)/nginx/nginx.conf:/etc/nginx/nginx.conf \
  -v $(pwd)/certbot/www:/var/www/certbot \
  nginx:alpine

sleep 5

# Generate SSL certificates
echo "Generating SSL certificate for all domains..."
docker run --rm \
  -v $(pwd)/certbot/www:/var/www/certbot \
  -v $(pwd)/certbot/conf:/etc/letsencrypt \
  certbot/certbot certonly \
  --webroot --webroot-path=/var/www/certbot \
  --email [YOUR_EMAIL] \
  --agree-tos --no-eff-email \
  -d consultinghub.xyz \
  -d www.consultinghub.xyz \
  -d admin.consultinghub.xyz \
  -d api.consultinghub.xyz

# Stop temporary nginx
docker stop temp-nginx && docker rm temp-nginx

# Create final nginx configuration
cat > nginx/nginx.conf << 'EOF'
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server backend:7000;
    }
    
    upstream frontend {
        server frontend:3000;
    }
    
    upstream admin {
        server admin:4000;
    }

    # Redirect HTTP to HTTPS
    server {
        listen 80;
        server_name consultinghub.xyz www.consultinghub.xyz admin.consultinghub.xyz api.consultinghub.xyz;

        location /.well-known/acme-challenge/ {
            root /var/www/certbot;
        }

        location / {
            return 301 https://$server_name$request_uri;
        }
    }

    # Main Frontend
    server {
        listen 443 ssl;
        server_name consultinghub.xyz www.consultinghub.xyz;

        ssl_certificate /etc/letsencrypt/live/consultinghub.xyz/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/consultinghub.xyz/privkey.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";

        location / {
            proxy_pass http://frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host $server_name;
        }
    }

    # Admin Panel
    server {
        listen 443 ssl;
        server_name admin.consultinghub.xyz;

        ssl_certificate /etc/letsencrypt/live/consultinghub.xyz/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/consultinghub.xyz/privkey.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";

        location / {
            proxy_pass http://admin;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host $server_name;
        }
    }

    # API Backend
    server {
        listen 443 ssl;
        server_name api.consultinghub.xyz;

        ssl_certificate /etc/letsencrypt/live/consultinghub.xyz/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/consultinghub.xyz/privkey.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host $server_name;
        }
    }
}
EOF

# Start all services
echo "Starting all services with SSL..."
docker-compose up -d

sleep 15
docker-compose ps

echo ""
echo "✅ Multi-service SSL setup complete!"
echo ""
echo "Your services are now accessible at:"
echo "  🌐 https://consultinghub.xyz (Frontend - User Panel)"
echo "  👤 https://admin.consultinghub.xyz (Admin Panel)"
echo "  🔌 https://api.consultinghub.xyz (Backend API)"
echo ""
echo "Test commands:"
echo "  curl -I https://consultinghub.xyz"
echo "  curl -I https://admin.consultinghub.xyz"
echo "  curl -I https://api.consultinghub.xyz"
```

Make it executable:
```bash
chmod +x multi-service-ssl-setup.sh
```

---

## Step 6: Deploy

Run the SSL setup script:
```bash
./multi-service-ssl-setup.sh
```

---

## Useful Commands

### Check Service Status
```bash
docker-compose ps
```

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f admin
docker-compose logs -f nginx
```

### Restart Services
```bash
docker-compose restart
```

### Rebuild and Restart
```bash
docker-compose up -d --build
```

### Stop Services
```bash
docker-compose down
```

---

## SSL Certificate Renewal

Certbot certificates expire every 90 days. Set up auto-renewal:

```bash
# Test renewal
docker run --rm \
  -v $(pwd)/certbot/www:/var/www/certbot \
  -v $(pwd)/certbot/conf:/etc/letsencrypt \
  certbot/certbot renew --dry-run

# Add to crontab for auto-renewal
crontab -e
```

Add this line:
```
0 3 * * * docker run --rm -v /root/nexadocs/certbot/www:/var/www/certbot -v /root/nexadocs/certbot/conf:/etc/letsencrypt certbot/certbot renew --quiet && docker restart nginx-proxy
```

---

## Security Best Practices

1. **Never commit credentials** to version control
2. Use **strong passwords** for all services
3. Keep **Docker images updated** regularly
4. Enable **firewall rules** (allow only 80, 443, SSH)
5. Use **environment-specific** `.env` files
6. Rotate **API keys and tokens** periodically
7. Monitor **logs** for suspicious activity
8. Implement **rate limiting** on Nginx
9. Use **secrets management** for production

---

## Troubleshooting

### SSL Certificate Issues
```bash
# Check certificate
docker exec nginx-proxy ls -la /etc/letsencrypt/live/consultinghub.xyz/

# Regenerate certificate
docker-compose down
./multi-service-ssl-setup.sh
```

### Container Won't Start
```bash
# Check logs
docker logs [container_name]

# Check disk space
df -h

# Check memory
free -h
```

### Network Issues
```bash
# Verify network
docker network ls
docker network inspect nexadocs_nexadocs_network

# Restart Docker
systemctl restart docker
```

---

## Architecture Notes

- **Backend**: Spring Boot application with PostgreSQL database
- **Frontend**: Next.js application for end users
- **Admin**: Next.js application for administrators
- **Nginx**: Reverse proxy with SSL termination
- **Certbot**: Automated SSL certificate management
- **Docker Network**: All services communicate via `nexadocs_network`

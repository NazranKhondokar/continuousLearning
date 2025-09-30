# Multi-Service SSL Setup for consultinghub.xyz

## Step 1: Update Namecheap DNS Records

Add these A records in Namecheap Advanced DNS:

```
Type    Host    Value               TTL
A       @       217.15.162.33       Automatic
A       www     217.15.162.33       Automatic  
A       admin   217.15.162.33       Automatic
A       api     217.15.162.33       Automatic
```

## Step 2: Create Master Docker Compose

Create a new `docker-compose.yml` in a master directory (like `/root/nexadocs/`):

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
    restart: always
    expose:
      - "7000"
    tty: true
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
      - ./NexaDoc_front/.env
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
      - ./NexaDoc_Admin/.env
    environment:
      - NODE_ENV=${NODE_ENV}
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

## Step 3: Create Multi-Service Nginx Config

Create `nginx/nginx.conf`:

```nginx
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

    # Main Frontend - consultinghub.xyz
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
            
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
    }

    # Admin Panel - admin.consultinghub.xyz  
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
            
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
    }

    # API Backend - api.consultinghub.xyz
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
            
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
    }
}
```

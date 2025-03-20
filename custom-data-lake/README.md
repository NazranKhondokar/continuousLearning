To configure **MinIO** in Docker, follow these steps:

---

### **Step 1: Pull the MinIO Docker Image**
Run the following command to pull the latest MinIO image:  
```sh
docker pull quay.io/minio/minio
```

---

### **Step 2: Create a Docker Volume (Optional)**
MinIO stores data in a persistent volume. To ensure data persistence, create a volume:  
```sh
docker volume create minio_data
```

---

### **Step 3: Run MinIO in Standalone Mode**
You can start a standalone MinIO container using the following command:  
```sh
docker run -d --name minio \
  -p 9000:9000 \
  -p 9090:9090 \
  -e "MINIO_ROOT_USER=admin" \
  -e "MINIO_ROOT_PASSWORD=admin123" \
  -v minio_data:/data \
  quay.io/minio/minio server /data --console-address ":9090"
```

- `-p 9000:9000`: Exposes MinIO API port  
- `-p 9090:9090`: Exposes MinIO web console  
- `-e MINIO_ROOT_USER=admin`: Sets the access key  
- `-e MINIO_ROOT_PASSWORD=admin123`: Sets the secret key  
- `-v minio_data:/data`: Maps a volume for persistence  
- `server /data --console-address ":9090"`: Starts MinIO with the web console  

---

### **Step 4: Access MinIO**
- **MinIO Web UI** → [http://localhost:9090](http://localhost:9090)  
  Login with:
  - Username: `admin`
  - Password: `admin123`
  
- **MinIO API** → `http://localhost:9000`  

---

### **Step 5: Run MinIO with Docker Compose (Optional)**
Create a `docker-compose.yml` file:
```yaml
version: '3.8'
services:
  minio:
    image: quay.io/minio/minio
    container_name: minio
    ports:
      - "9000:9000"
      - "9090:9090"
    environment:
      MINIO_ROOT_USER: admin
      MINIO_ROOT_PASSWORD: admin123
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9090"

volumes:
  minio_data:
```

Start the container:
```sh
docker-compose up -d
```

---

### **Step 6: Install MinIO Client (mc) (Optional)**
To interact with MinIO via CLI, install the MinIO client:  
```sh
curl -O https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/
```

Add MinIO to the client:
```sh
mc alias set local http://localhost:9000 admin admin123
```

Create a bucket:
```sh
mc mb local/mybucket
```

---

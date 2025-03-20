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
To configure **Apache Iceberg** in Docker on Linux, follow these steps:

---

## **Step 1: Install Docker & Docker Compose**
Ensure **Docker** and **Docker Compose** are installed. If not, install them using:

```sh
sudo apt update && sudo apt install -y docker.io docker-compose
```

Verify installation:
```sh
docker --version
docker-compose --version
```

---

## **Step 2: Set Up Required Services**
Apache Iceberg requires a **Metastore (Hive)** and a **Query Engine (Trino, Spark, or Flink)**.  
Here, we will use **Trino + MinIO + PostgreSQL (Hive Metastore)**.

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

  postgres:
    image: postgres:14
    container_name: postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: hive
      POSTGRES_PASSWORD: hive
      POSTGRES_DB: metastore
    volumes:
      - postgres_data:/var/lib/postgresql/data

  hive-metastore:
    image: apache/hive:4.0.0-alpha-2
    container_name: hive-metastore
    depends_on:
      - postgres
    ports:
      - "9083:9083"
    environment:
      SERVICE_NAME: metastore
      HIVE_METASTORE_URIS: thrift://hive-metastore:9083
      METASTORE_DB_TYPE: postgres
      METASTORE_DB_HOST: postgres
      METASTORE_DB_NAME: metastore
      METASTORE_DB_USER: hive
      METASTORE_DB_PASSWORD: hive
    command: /opt/hive/bin/hive --service metastore

  trino:
    image: trinodb/trino:latest
    container_name: trino
    depends_on:
      - hive-metastore
      - minio
    ports:
      - "8081:8080"
    volumes:
      - ./trino-config:/etc/trino

volumes:
  minio_data:
  postgres_data:
```

---

## **Step 3: Configure Trino for Iceberg**
Create a directory `trino-config` and add a configuration file for Iceberg.

### **trino-config/catalog/iceberg.properties**
```ini
connector.name=iceberg
catalog.type=hive
hive.metastore.uri=thrift://hive-metastore:9083
iceberg.catalog.type=hive
hive.s3.aws-access-key=admin
hive.s3.aws-secret-key=admin123
hive.s3.endpoint=http://minio:9000
hive.s3.path-style-access=true
```

---

## **Step 4: Start the Docker Containers**
Run the following command to start all services:

```sh
docker-compose up -d
```

---

## **Step 5: Access Trino Web UI**
- Open Trino in your browser → [http://localhost:8081](http://localhost:8081)

---

## **Step 6: Create an Iceberg Table in Trino**
Run the following SQL command inside Trino:
```sql
CREATE SCHEMA iceberg.iceberg_db 
WITH (location = 's3a://iceberg-data/');
```

Then, create a table:
```sql
CREATE TABLE iceberg.iceberg_db.customers (
    id BIGINT,
    name VARCHAR,
    email VARCHAR
) WITH (
    format = 'PARQUET'
);
```

---

## **Step 7: Verify Iceberg Table**
Run:
```sql
SELECT * FROM iceberg.iceberg_db.customers;
```

---

Now **Apache Iceberg** is configured in **Docker on Linux** using **Trino, Hive Metastore, and MinIO**. 🚀

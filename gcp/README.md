Running a **GCP Cloud Run service with a database connection** requires some considerations because Cloud Run services are **stateless, containerized applications** that run on Google’s fully managed serverless platform. The main challenge is securely and reliably connecting to your database from Cloud Run.

---

### ✅ 1. **Choose Your Database**

Common options:

* **Cloud SQL** (MySQL, PostgreSQL, SQL Server)

We use **Cloud SQL (PostgreSQL or MySQL)**

---

### ✅ 2. **Set Up Your Database**

* Create a **Cloud SQL instance** in GCP:

  * Go to **Cloud SQL → Create instance**.
  * Choose **PostgreSQL/MySQL**.
  * Configure:

    * Enable **Private IP** (recommended for security).
    * Set **user**, **password**, **database name**.
  * Enable **Cloud SQL Admin API**.

---

### ✅ 3. **Connect Using Cloud SQL Auth Proxy (Best Practice)**

The safest way to connect is through the **Cloud SQL Auth Proxy**, which handles:

* IAM authentication
* Secure connection (no hardcoded passwords)
* Automatic SSL

#### **Option A: Use Cloud SQL Auth Proxy in Cloud Run (recommended)**

* Enable **Cloud SQL Admin API**.
* In Cloud Run deployment:

  * Under **Connections**, click **Cloud SQL connections** and add your Cloud SQL instance.
* This automatically mounts the Unix socket `/cloudsql/PROJECT:REGION:INSTANCE`.

Example **connection string**:

```
PostgreSQL:
DATABASE_URL=postgresql://USER:PASSWORD@/DB_NAME?host=/cloudsql/PROJECT:REGION:INSTANCE

MySQL:
DATABASE_URL=mysql://USER:PASSWORD@unix(/cloudsql/PROJECT:REGION:INSTANCE)/DB_NAME
```

---

### ✅ 4. **Connect with DBeaver in Ubuntu**

* Step 1:

```
nazran@naxalinx:~$ curl -o cloud-sql-proxy https://storage.googleapis.com/cloud-sql-connectors/cloud-sql-proxy/v2.18.1/cloud-sql-proxy.linux.amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 34.8M  100 34.8M    0     0  2562k      0  0:00:13  0:00:13 --:--:-- 2823k
```
* Step 2:
```
nazran@naxalinx:~$ chmod +x cloud-sql-proxy
```
* Step 3: It will open a new window for login with a Google account
```
nazran@naxalinx:~$ gcloud auth application-default login
Your browser has been opened to visit:

    https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=764086051850-6qr4p6gpi6hn506pt8ejuq83di341hur.apps.googleusercontent.com&redirect_uri=http%3A%2F%2Flocalhost%3A8085%2F&scope=openid+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fsqlservice.login&state=fBedcMR2qTJyuX0SGFNPhNVOZed2Ju&access_type=offline&code_challenge=KJDagC3yBae7kEygDGt5txeAcdbgjl0raH3BSWc1Nfg&code_challenge_method=S256


Credentials saved to file: [/home/nazran/.config/gcloud/application_default_credentials.json]

These credentials will be used by any library that requests Application Default Credentials (ADC).

Quota project "bitcoin-canvas-428314" was added to ADC which can be used by Google client libraries for billing and quota. Note that some services may still bill the project owning the resource.

```
* Step 4:
```
nazran@naxalinx:~$ ./cloud-sql-proxy bitcoin-canvas-428314:us-central1:postgres-production --port 5432
2025/08/21 15:00:32 Authorizing with Application Default Credentials
2025/08/21 15:00:32 [bitcoin-canvas-428314:us-central1:postgres-production] Listening on 127.0.0.1:5432
```
* Step 5: After connecting to DBeaver with the below credentials, then get logs like
```
DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=your_database_name
DB_USER=your_database_user
DB_PASSWORD=your_database_password
```
```
2025/08/21 15:00:32 The proxy has started successfully and is ready for new connections!
2025/08/21 15:03:32 [bitcoin-canvas-428314:us-central1:postgres-production] Accepted connection from 127.0.0.1:55070
2025/08/21 15:03:40 [bitcoin-canvas-428314:us-central1:postgres-production] client closed the connection
2025/08/21 15:04:07 [bitcoin-canvas-428314:us-central1:postgres-production] Accepted connection from 127.0.0.1:60994
2025/08/21 15:04:11 [bitcoin-canvas-428314:us-central1:postgres-production] Accepted connection from 127.0.0.1:32774
```
---

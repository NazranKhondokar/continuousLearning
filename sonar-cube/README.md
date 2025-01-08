### 1. **Install SonarQube Server in Linux**

#### **Prerequisites**
- **Java JDK**: Ensure Java 11 or 17 is installed (SonarQube doesn’t support Java 8 or older).
- **Database**: Set up a supported database (e.g., PostgreSQL, MySQL, or Oracle) for production use. For testing, the default H2 database is sufficient.
- **Memory**: Allocate at least 2GB of RAM for the server.

#### **Download and Install**
1. Download the latest SonarQube version from the [official site](https://www.sonarsource.com/products/sonarqube/).
2. Extract the downloaded file to a directory of your choice.
3. Navigate to the `conf/sonar.properties` file:
   - Configure your database connection if using an external database:
     ```properties
     sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
     sonar.jdbc.username=your_db_user
     sonar.jdbc.password=your_db_password
     ```
4. Start SonarQube:
     ```bash
      nazran@naxalinx:~/sonarqube-9.9.8.100196/bin$ ls
      linux-x86-64  macosx-universal-64  windows-x86-64  winsw-license
      nazran@naxalinx:~/sonarqube-9.9.8.100196/bin$ cd linux-x86-64/
      nazran@naxalinx:~/sonarqube-9.9.8.100196/bin/linux-x86-64$ ls
      sonar.sh
      nazran@naxalinx:~/sonarqube-9.9.8.100196/bin/linux-x86-64$ ./sonar.sh start
      /usr/bin/java
      Starting SonarQube...
      Started SonarQube.
     ```

5. Access the server at `http://localhost:9000` (default credentials: `admin/admin`).

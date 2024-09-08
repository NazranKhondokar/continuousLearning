## Kill port when stop button not work
```
npx kill-port <port no>
```

## Increase heap size on `./gradlew`
```bash
./gradlew bootRun --no-daemon -Dorg.gradle.jvmargs="-Xms512m -Xmx2048m"
```
- If this command does not work then increase the request size on `application.yml` in `Spring boot`
```bash
spring:
  servlet:
    multipart:
      max-file-size: 1GB
      max-request-size: 2GB
```

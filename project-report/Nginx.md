```bash
# Frontend (User Panel)
https://consultinghub.xyz:9443

# Admin Panel
https://admin.consultinghub.xyz:9443

# Backend API
https://api.consultinghub.xyz:9443
```

Three options:
## Option 1: Use netbill360 nginx consultinghub also
You can add your consultinghub.xyz domains to the existing `netbill360_nginx_dev` container. Both applications would share the same nginx on ports 80/443.

This way:
- `consultinghub.xyz` → accessible without port numbers
- Your netbill360 domains → still work normally
- Everything runs on standard ports 80/443

## Option 2: Stop netbill360 nginx temporarily
Since `netbill360_nginx_dev` is using ports 80/443, you can stop it and use those ports for consultinghub:

```bash
# Stop netbill360 nginx
docker stop netbill360_nginx_dev

# Then I'll update your scripts to use ports 80/443
```

## Option 3: Run netbill360 on different ports
Move netbill360 to alternative ports (like 8080/8443) so consultinghub can use 80/443.

---

**Which option do you prefer?**


### Install on Ubuntu
- Update the system
```bash
sudo apt update
sudo apt upgrade
```
- Install Redis
```bash
sudo apt install redis-server
```
- Ensure the server is running
```bash
sudo systemctl status redis-server
```
- If status is `inactive` the `start` the service
```bash
sudo systemctl start redis-server
```
- Verify the installation
```bash
redis-cli ping
```
- Response will be
```bash
PONG
```

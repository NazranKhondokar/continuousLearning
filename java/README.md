## Update `$JAVA_HOME` on Ubuntu
- Check the available jdks
```bash
update-alternatives --config java
```
- Copy the selection path without `/bin/java`
```bash
/usr/lib/jvm/java-17-openjdk-amd64
```
- Edit the file
```bash
nano ~/.bashrc
```
- Add the following line
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```
- Save the file (`Ctrl + O`, then `Enter`) and exit (`Ctrl + X`)
- Apply the changes by running
```bash
source ~/.bashrc
```
- Ensure that the JAVA_HOME variable is now set correctly by running
```bash
echo $JAVA_HOME
```
- It should output `/usr/lib/jvm/java-17-openjdk-amd64`

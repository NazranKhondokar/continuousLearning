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

# Installation on Windows
To install Java via the Windows Command Prompt, you'll need to download the Java Development Kit (JDK) and set it up. Here’s a step-by-step guide:

### Step 1: Download the JDK
1. Go to [OpenJDK page](https://jdk.java.net/) for a free version.
2. Choose the version you need and download the Windows installer (usually a `.exe` file).

### Step 2: Install the JDK
1. Run the downloaded installer.
2. Follow the installation prompts. By default, it installs in `C:\Program Files\Java\jdk-<version>`.

### Step 3: Set Environment Variables
1. **Open Command Prompt**: Press `Win + R`, type `cmd`, and hit `Enter`.
2. **Set JAVA_HOME**:
   - Type the following command (replace `<version>` with your installed JDK version):
     ```bash
     setx JAVA_HOME "C:\Program Files\Java\jdk-<version>"
     ```
3. **Update the PATH Variable**:
   - Add the JDK `bin` directory to your PATH by typing:
     ```bash
     setx PATH "%PATH%;%JAVA_HOME%\bin"
     ```

### Step 4: Verify the Installation
1. Close the Command Prompt and reopen it to ensure the environment variables are updated.
2. Type the following command to check the Java installation:
   ```bash
   java -version
   ```
   You should see the installed version of Java.

### Step 5: Test the Installation
You can create a simple Java program to test if everything works:
1. Open a text editor and write a simple Java program (e.g., `HelloWorld.java`):
   ```java
   public class HelloWorld {
       public static void main(String[] args) {
           System.out.println("Hello, World!");
       }
   }
   ```
2. Save it in a directory (e.g., `C:\Java`).
3. Open the Command Prompt, navigate to the directory, and compile the program:
   ```bash
   cd C:\Java
   javac HelloWorld.java
   ```
4. Run the program:
   ```bash
   java HelloWorld
   ```

If you see "Hello, World!" printed on the screen, you’ve successfully installed and set up Java!

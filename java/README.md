## Update `$JAVA_HOME` on Ubuntu
- Check the available jdks
```bash
nazran@naxalinx:~$ sudo update-alternatives --config java
```

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

# Installation on MAC using `Homebrew`
To install Java 21 on a Mac using Homebrew, follow these steps:

### Step 1: Install Homebrew (If Not Installed)
If you haven't already installed Homebrew, open Terminal and enter:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Step 2: Install OpenJDK 21 with Homebrew
1. Open Terminal and run the following command to install OpenJDK 21:
   ```bash
   brew install openjdk@21
   ```

2. After the installation completes, Homebrew will display the path to the Java 21 installation, typically something like:
   ```
   /opt/homebrew/opt/openjdk@21
   ```

### Step 3: Configure `JAVA_HOME`
To ensure that Java 21 is recognized as the default Java version, set the `JAVA_HOME` environment variable.

1. Add the following line to your shell profile file:
   - **For Zsh** (default on macOS Catalina and newer): edit `~/.zshrc`.
   - **For Bash**: edit `~/.bash_profile` or `~/.bashrc`.
   
   ```bash
   export JAVA_HOME=/opt/homebrew/opt/openjdk@21
   export PATH="$JAVA_HOME/bin:$PATH"
   ```
   - Save the file (`Ctrl + O`, then `Enter`) and exit (`Ctrl + X`)

2. Reload the profile configuration by running:
   ```bash
   source ~/.zshrc  # or source ~/.bash_profile, depending on your shell
   ```

### Step 4: Verify the Installation
Finally, confirm that Java 21 is installed and set as the default version:
```bash
java -version
```

You should see output like:
```
java version "21.0.x" 2023-09-19 LTS
```

Java 21 is now installed and configured on your Mac using Homebrew!

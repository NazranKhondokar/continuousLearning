## Setup using `Homebrew`
- Installation Homebrew on Mac:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- Installation Node:
```bash
brew install node
```

- Installation Angular CLI:
```bash
brew install angular-cli
```

- Check Angular CLI:
```bash
ng version
```

- Then move to any directory and create a new project:
```bash
sudo ng new <projectname>
```

- Then run project:
```bash
sudo ng serve
```

- To avoid using sudo every time for ng serve on macOS, you can modify the file permissions to allow your user account to execute the command without requiring elevated privileges.
```bash
which ng
```
This will output the path to the ng command, which should be something like `/opt/homebrew/bin/ng`.
Now, you need to change the file permissions of the ng command to allow your user account to execute it without requiring sudo. You can do this by running:

```bash
sudo chown -R $(whoami) /opt/homebrew/bin/ng
sudo chmod -R u+rwx /opt/homebrew/bin/ng
```

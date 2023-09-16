## Build PDF when run `.tex` file on VS Code
- Open the Terminal app on macOS
- 

- Instal mactex
```
brew install --cask mactex
```

- If the command is not found, We have to update the `$PATH`
```
zsh: command not found: mactex
```
- First `echo $PATH`
```
/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/homebrew/bin:/opt/homebrew/bin
```
- add the `/Library/TeX/texbin` directory to the `$PATH` variable. Edit the file `$HOME/.bash_profile`, so type:
```
vi $HOME/.bash_profile
```
- Append the following export command
```
export PATH="$PATH:/Library/TeX/texbin
```
- 

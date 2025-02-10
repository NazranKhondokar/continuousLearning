## To create a folder in a GitHub repository

- Navigate to Your Repository:
  Go to the main page of your repository on GitHub.
  
- Click on `Add File` and select `Create New File` or `Upload Files`:
  You can find the `Add file` button near the top right of your repository's main page.
  
- Specify the Folder Name:
  In the file name field, enter the folder name followed by a forward slash / and then the file name (e.g., `folder_name/file_name.txt`).
  GitHub will recognize the forward slash as an indication to create a folder.
  
- Commit New File:
  Below the file name field, you'll find an option to add a commit message. Type in a meaningful message to describe the changes you're making.
  
- Commit Changes:
  Finally, click on the `Commit new file` or `Commit changes` button at the bottom.

## Login for a specific remote git repository using token

```bash
git remote set-url origin https://NazranKhondokar:<token>@github.com/NazranKhondokar/nazrankhondokar.github.io.git
```

## Generating a new GPG key

  Follow those links for detail:
  - https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key
  - https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
```bash
ssh-keygen -t ed25519 -C "nazran91@gmail.com"
```
```bash
cd ~/.ssh
```
```bash
eval "$(ssh-agent -s)"
```
```bash
ssh-add ~/.ssh/id_ed25519
```
```bash
cat ~/.ssh/id_ed25519.pub
```

### **Find Your GPG Key ID**:

Run this command to list your GPG keys:
   ```
   gpg --list-secret-keys --keyid-format=long
   ```
Look for the **sec** line and note the long key ID (e.g., `ABCD1234EFGH5678`).

### **Set the GPG Key in Git**:
   ```
   git config --global user.signingkey ABCD1234EFGH5678
   ```

### **Enable Commit Signing**:
   ```
   git config --global commit.gpgsign true
   ```

### Verify Your Configuration:
To check if the signing key is set:
```
git config --global --get user.signingkey
```

## Author

- [Nazran Khondokar][author]

<!-- Definitions -->
[author]: https://www.linkedin.com/in/nazran91/

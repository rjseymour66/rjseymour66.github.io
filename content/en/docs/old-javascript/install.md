---
title: "Installation"
weight: 10
description: >
  Installing nvm, Node.js, and npm.
---

### Install nvm

Install nvm to help manage your Node.js versions.

1. Go to the [nvm-windows release page](https://github.com/coreybutler/nvm-windows/releases) on GitHub.
2. The latest release is at the top. Find the **Assets** section, and select **nvm-setup.zip**.
3. Open the Windows File Explorer and navigate to `/Downloads`.
4. Double-click on the **nvm-setup.zip** folder.
5. Double-click on **nvm-setup.exe**.
6. Accept the license agreement and select **Next**.
7. On **Select Destination Location**, select **Next** to use the default directory.
8. On **Set Node.js Symlink**, select **Next** to create a symlink to the default directory.
9. Select **Install**.
10. After the installation completes, open a PowerShell or git-bash shell and verify the installation was successful with the following command:
    ```bash
    $ nvm version
    1.1.9
    ```
    If the installation was successful, the nvm version displays beneath the command.

### Install Node.js

You must complete [Installing nvm](#installing-nvm) before you can install Node.js.

> Every other installation section uses a git-bash for Windows shell. The following instructions are run in Windows PowerShell with Administrator privileges.


1. Click on the Windows Start button and type **PowerShell**.
2. Right-click on PowerShell, and select **Run as administrator** from the context menu.
3. View the available Node.js versions with the following command:
   ```bash
   > nvm list available

   |   CURRENT    |     LTS      |  OLD STABLE  | OLD UNSTABLE |
   |--------------|--------------|--------------|--------------|
   |    17.8.0    |   16.14.2    |   0.12.18    |   0.11.16    |
   |    17.7.2    |   16.14.1    |   0.12.17    |   0.11.15    |
   |    17.7.1    |   16.14.0    |   0.12.16    |   0.11.14    |
   ...

   This is a partial list. For a complete list, visit https://nodejs.org/en/download/releases
   ```
4. Install the latest stable LTS (long-term support) version:
   ```bash
   > nvm install 16.14.2
   ```
5. Verify the installation with the following command:
   ```bash
   > nvm ls
   
       17.7.1
       16.14.2
     * 16.14.0 (Currently using 64-bit executable)
   ```
6. If the asterisk does not indicate that you are using the correct LTS version, select the correct version with `nvm use`:
   ```bash
   > nvm use 16.14.2
   Now using node v16.14.2 (64-bit)
   ```
7. Verify that you are using the correct version:
   ```bash
   > nvm ls

      17.7.1
   * 16.14.2 (Currently using 64-bit executable)
      16.14.0
   ```

### Verify npm

When you install a Node.js version with `nvm`, npm is installed automatically. Verify your `npm` version with the following command:

```bash
$ npm --version
8.5.0
```
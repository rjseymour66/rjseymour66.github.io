
---
title: "Documentation"
linkTitle: "Documentation"
weight: 20
menu:
  main:
    weight: 20
---

{{% pageinfo %}}
This is a placeholder page that shows you how to use this template site.
{{% /pageinfo %}}


This section is where the user documentation for your project lives - all the information your users need to understand and successfully use your project. 

For large documentation sets we recommend adding content under the headings in this section, though if some or all of them don’t apply to your project feel free to remove them or add your own. You can see an example of a smaller Docsy documentation site in the [Docsy User Guide](https://docsy.dev/docs/), which lives in the [Docsy theme repo](https://github.com/google/docsy/tree/master/userguide) if you'd like to copy its docs section. 

Other content such as marketing material, case studies, and community updates should live in the [About](/about/) and [Community](/community/) pages.

Find out how to use the Docsy theme in the [Docsy User Guide](https://docsy.dev/docs/). You can learn more about how to organize your documentation (and how we organized this site) in [Organizing Your Content](https://docsy.dev/docs/best-practices/organizing-content/).




The Vertica documentation team authors content in [Markdown](https://www.markdownguide.org/getting-started/) and publishes the documentation with Hugo, a static site generator that renders Markdown files as HTML.

For guidance about Markdown elements in Hugo, see [Hugo Markdown Support](https://www.markdownguide.org/tools/hugo/).

  
Unless otherwise noted, the following instructions use [git for Windows](https://gitforwindows.org/) for all terminal commands.  


## Install a text editor

Because Markdown is simple and lightweight, you can author content with a text editor rather than a robust authoring tool that drives you crazy (looking at you, Madcap Flare). Vertica documentation supports [Microsoft Visual Studio Code](https://code.visualstudio.com/) (VSCode) as the offical text editor.


VSCode includes an [integrated terminal](https://code.visualstudio.com/docs/terminal/basics) that you can use to execute commands from within your workspace. In some environments, building Hugo with the integrated terminal might spawn subprocesses that consume system memory and decrease system performance.

If you encounter this issue, install a more stable version of VSCode. The following steps install version 1.72:
1. In a web browser, go to The Visual Studio Code updates for [September 2022 (version 1.72)](https://code.visualstudio.com/updates/v1_72).
2. Directly under the section title, there is a **Downloads** section that lists the available downloads by operating system. Select the download for your system (Windows users should select the User option).  
   A Visual Studio Code installer begins downloading.
3. Open the installer and follow the instructions to complete the installation.


## Install Go

Hugo and Docsy use the Go programming language for templating and dependency management. Follow the instructions on [Download and install](https://go.dev/doc/install) to install Go on Windows with the .msi (Microsoft Windows Installer) program.

## Install Hugo

The following sections describe how to install Hugo with the Chocolatey package manager, or how to download the Hugo binary and add it to your path.

Hugo provides two downloads: the standard package and the extended package. [Docsy requires](#install-docsy-dependencies) the extended package because it includes tools to run local builds and supports [SCSS](https://sass-lang.com/documentation/syntax), a superset of CSS with additional syntax features.

### Quick install with Chocolatey (Recommended)

If you use the [Chocolatey](https://chocolatey.org/) package manager, enter the following command to install Hugo:

```bash
$ choco install hugo-extended --version=0.99.1 -confirm
```
If you have an older version of Hugo, enter the following command to upgrade:

```bash
$ choco upgrade hugo-extended --version=0.99.1 -confirm
```
### Install Hugo Manually

To install Hugo manually, you must place the Hugo binary in a directory under your `/C:` drive and add that directory to your Windows path.

The following steps install the extended package. They are adapted from the [official Hugo docs](https://gohugo.io/getting-started/installing/#windows):

1. Navigate to the directory where you want to store the Hugo binary and associated files. This tutorial uses the `/C:` drive.
   ```bash
   $ cd /C
   ```
2. Create directories for the Hugo binary files:
   ```bash
   $ mkdir Hugo
   $ mkdir Hugo/bin
   ```
   The previous command creates:
   - The top-level `/Hugo` directory.
   - A `/Hugo/bin` directory that stores the Hugo executable and additonal files.

3. Navigate to the [Hugo Releases GitHub page](https://github.com/gohugoio/hugo/releases/tag/v0.99.1).
4. In the **Assets** section, download the correct `hugo_extended_0.99.1_*` zip file for your OS. For example, Windows users should download `hugo_extended_0.99.1_Windows-64bit.zip`.
5. Open the Windows File Explorer and navigate to `/Downloads`.
6. Locate the `hugo_extended_...zip` file. Right-click the file and select **Extract All...** from the context menu.
7. In the **Select a Destination and Extract Files** window, click **Browse** and select the `C:/Hugo/bin` directory.
8. Verify that Windows extracted the following files into the `C:/Hugo/bin` directory:
   - **hugo.exe**
   - **LICENSE**
   - **README.md**
9. Add the `C:/Hugo/bin` directory to your Windows PATH to use Hugo commands. For details, see [For Windows 10 Users](https://gohugo.io/getting-started/installing/#for-windows-10-users) in the official Hugo documentation.

## Install Docsy dependencies

[Docsy](https://www.docsy.dev/docs/) is a Hugo theme that is available in a GitHub repository. Before you can use it in a Hugo project, you must install Javascript runtime tools and PostCSS to build and render the site locally.

The following installation instructions are adapted from [Install nvm-windows, node.js, and npm](https://docs.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-windows#install-nvm-windows-nodejs-and-npm) in the Microsoft documentation.

### Prerequisites

Docsy requires the following to build and render Markdown content:
- node volume manager (nvm): A package manager for Node.js and npm
- Node.js: A Javascript runtime environment
- Node Package Manager (npm): A package manager for Node.js
- PostCSS: A collection of Javascript packages that create CSS assets

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


Every other installation section uses a git-bash for Windows shell. The following instructions are run in Windows PowerShell with Administrator privileges.



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

### Install PostCSS

After you install nvm, Node.js, and npm, install PostCSS. Docsy uses PostCSS to create the final CSS assets. Use the following commands to install the PostCSS requirements:
```bash
$ npm install -D autoprefixer
$ npm install -D postcss-cli
$ npm install -D postcss
```

## Clone the repo

After you install [Go](#install-go), [Hugo](#install-hugo), and the [Docsy dependencies](#install-docsy-dependencies), you can author a Hugo/Docsy project and build it locally.

The following steps clone the [Vertica Style Guide repo](http://git.verticacorp.com/projects/DOCS/repos/style-guide-hugo/browse), but the steps apply to any repository containing a Hugo/Docsy project:

1. Navigate to `C:/doc`, or wherever you want to clone this repository:
   ```bash
   $ cd /C/doc
   ```
2. Clone the Bitbucket repository:
   ```bash
   $ git clone http://git.verticacorp.com/scm/docs/style-guide-hugo.git
   ```
3. `cd` into the `style-guide-hugo` directory you just cloned:
   ```bash
   $ cd style-guide-hugo
   ```
4. Enter `hugo serve`. Hugo downloads any dependencies with [Hugo Modules](https://gohugo.io/hugo-modules/), builds the site, and then serves it on `localhost:1313`:
   ```bash
   $ hugo serve
   hugo: collected modules in 511 ms
   Start building sites …
   hugo v0.98.0-165d299cde259c8b801abadc6d3405a229e449f6+extended windows/amd64 BuildDate=2022-04-28T10:23:30Z VendorInfo=gohugoio
   WARN 2022/07/08 09:29:28 .File.UniqueID on zero object. Wrap it in if or with: {{ with .File }}{{ .UniqueID }}{{ end }}

                     | EN
   -------------------+-----
   Pages            | 24
   Paginator pages  |  0
   Non-page files   |  0
   Static files     | 40
   Processed images |  0
   Aliases          |  0
   Sitemaps         |  1
   Cleaned          |  0

   Built in 743 ms
   ...
   Web Server is available at //localhost:1313/ (bind address 127.0.0.1)
   Press Ctrl+C to stop
   ```
5. Enter `localhost:1313` in your web browser to view the project. Hugo re-renders the site each time you save your work.



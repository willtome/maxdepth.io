---
layout: post
title: Visual Studio Code for Windows
comment_id: 2
categories: [tech]
tags: [tech]
---

Every few years I find myself transitioning from one text editor to another. Starting with NotePad++ I moved to [Sublime Text](http://www.sublimetext.com), then [Atom](https://atom.io), and more recently to [Visual Studio Code](https://code.visualstudio.com). Just for clarification, I run a Mac but one reason I've liked VS Code so much is because of how it works across platforms, especially Windows. If you're really just interested in the setup, scroll down a little. Before I get to that though, I want to highlight a few reasons why VS Code has become one of my favorites.

* **Cross Platform** - VS Code works more or less equally across Windows, Mac, and Linux giving a consist interface and experience despite your operating system. Plus, it being a Microsoft product makes it great for Windows users.

* **Git Integration** - Git is hard. Especially when you are new and especially when you don't feel at home in a shell. Git is a HUGE part of an Ansible code workflow or really any modern "devops" practices. The graphical git controls help make this easier for newcomers.

* **Integrated Shell** - VS Code offers an integrated shell that can be opened in the interface reducing the number of windows and switching you need to do. You can customize what shell is actually used too (eg. open a bash shell on Windows).

## Visual Studio Code Install/Setup

Here are the instructions I recommend to get VS Code installed and configured on a Windows system.

1. Download and Install VS Code from [code.visualstudio.com](https://code.visualstudio.com)
![VS Code](/images/posts/1.png)

2. Run the installer
![Installer](/images/posts/2.png)

3. Download and Install Git for Windows from [gitforwindows.org](https://gitforwindows.org)

Visual Studio Code is now installed. Next we will install Git and configure the integration with VS Code. 

4. Run the Installer for Git
    1. Accept the license ![license](/images/posts/4.png)
    2. Select install Destination and __WRITE THIS DOWN__. We will need it later ![destination](/images/posts/5.png)
    3. Select Components. These can be left as defaults ![components](/images/posts/6.png)
    4. Menu Options. These can be left as defaults ![menu](/images/posts/7.png)
    5. Default Editor. Change to Visual Studio Code ![editor](/images/posts/8.png)
    6. Adjust your PATH. Leave as default ![PATH](/images/posts/9.png)
    7. SSL Library. Leave as default ![openssl](/images/posts/10.png)
    8. Checkout Style. Leave as default ![checkout](/images/posts/11.png)
    9. Terminal Style. Leave as default ![terminal](/images/posts/12.png)
    10. Extra Option. Leave as default ![extras](/images/posts/13.png)

5. Open a file explorer Windows and navigate to where Git was installed (`C:\Program Files\Git` by default). Look for a directory called `bin`. Inside `bin` you should find `bash.exe`. Make a note of the full path. By default, this should be `C:\Program Files\Git\bin\bash.exe` ![programs](/images/posts/17.png)

6. Open VS Code, navigate to `File > Preferences > Settings`. In the Settings tab, search for `integrated shell`. The setting we are looking for should look like "Terminal > Integrated > Shell:Windows" and have a path to `powershell.exe`. Replace the path to `powershell.exe` with the path to `bash.exe` from the previous step. Make sure to do a `File > Save` before closing. ![shell](/images/posts/16.png)

7. Close and reopen VS Code. Select `View > Terminal` and the bottom section of the interface should open a terminal to a bash prompt.

At this point, VS Code is installed and configured to use a bash shell. If you'd like to use the git integration, follow the next steps for creating your ssh keys. 

8. In the terminal, type `ssh-keygen`. You will be given several prompts. Proceed by pressing enter for each of them to accept defaults. ![ssh-keygen](/images/posts/14.png)

9. Now that you have a key pair created, `cat` your public key with the command `cat ~/.ssh/id_rsa.pub`. Copy the blob starting with `ssh-rsa`. ![public key](/images/posts/15.png
)

10. Finally, add the public key that you copied to your Git server. 

    On [Github](https://github.com) you can find this by clicking your avatar in the top right corner, selecting `Settings`, then selecting `SSH and GPG Keys` from the left side.

    In [BitBucket](https://bitbucket.org), click your avatar in the bottom left corner, choose `Settings`, then look for the `SSH Keys` section under Security.

    **NOTE**: if you are new to using SSH keys, it is ok to share your public key (`id_rsa.pub`) with others but DO NOT share your private key (`id_rsa`). Sharing your private key is the equivalent of giving away your password.
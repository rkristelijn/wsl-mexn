# Overview

Below is a comparison from one of the resources I used. I don't think the comparison is done using the same hardware, but the big take away is that Windows without WSL is a no-go, and Windows WSL underperforms for Linux on bare metal and even Virtual Linux on a Windows Guest. I still have to do this comparison myself using my Udoo with dualboot. 

System | Performance 
--- | --- 
Mac Mini 2012 Core i5 10GB RAM | 217.6 sec 
Windows WSL | 335 sec
Linux Mint Bare Metal | 182.4 sec
VirtualBox | 124 sec

# Versions

Item | Version
--- | ---
MongoDB | 4.0 Community Edition
NPM | 6.4.1
Node | 11.1.0
WSL | Windows 10 Enterprise OS v 1803 build 17134.345
vscode | 1.18.2
git | v2.11.0
gitkraken | 4.0.6
Linux | Debian GNU/Linux 9 (stretch)

# Credits

# End result
![img][result.png]

# Windows 10 Fall Creators Update - Installing Node.js on Windows Subsystem for Linux (WSL)

Windows just released the [windows subsystem for linux](https://msdn.microsoft.com/en-us/commandline/wsl/about) feature to the public with its latest windows fall creator update, if you are not familiar with this feature it allows you to run linux binaries natively on windows - [F.A.Q](https://msdn.microsoft.com/en-us/commandline/wsl/faq).

## Enabling WSL
The feature is not enabled by default and you need to activate it, you can do it via powershell (with admin rights):
```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```
Or you can open: Control-Panel -> Programs -> Turn Windows feature on ro off, and click the "windows subsystem for linux (beta)" button.

Now to install your linux version, again two options, you can either install from Windows store (search linux) or you can turn on developer mode and open it via the command line:
Open Settings -> Update and Security -> For developers, and Select the "Developer Mode" radio button.

## Installing linux
For those who are not familiar with linux, fragmentation if part of the fun and there a lot of linux distributions and package management systems for it, the default installation in WSL is ubuntu and I recommend to keep it as it one of the most popular distribution around. If you need different distribution the easiest why is to install it from the windows store.

Search for Linux, you can use any distro, but I use Debian (Ubuntu is derived from Debian)

Run CMD.exe and type:
```
bash
```

When you first run the command you'll encounter the cli based installer wizard, the ubuntu image will automatically download and you'll be prompt to enter username and password. If you found yourself in need to reset/reinstall the ubuntu machine you can use `lxrun.exe`:
```
CMD> lxrun
Performs administrative operations on the LX subsystem

Usage:
    /install - Installs the subsystem
        Optional arguments:
            /y - Do not prompt user to accept
    /uninstall - Uninstalls the subsystem
        Optional arguments:
            /full - Perform a full uninstall
            /y - Do not prompt user to accept
    /setdefaultuser - Configures the subsystem user that bash will be launched as. If the user does not exist it will be created.
        Optional arguments:
            username - Supply the username
            /y - If username is supplied, do not prompt to create a password
    /update - Updates the subsystem's package index
```

That's it, you're running debian on windows!

[reference](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide)

## Installing Node.js

First We'll start by updating linux, for those of you that are not familiar with linux this require running the process as `root` by add the `sudo` command before the command we need to execute:
```
sudo apt-get update
sudo apt-get upgrade
```
You'll need to enter your user password to proceed, if you prompt just press y (or yes) to continue.

If you don't want to type the password every single time when you need to use sudo, then better you add the following line (at the end) to `sudo visudo`: (replace `username` with your username)

```
username    ALL=(ALL:ALL) NOPASSWD:ALL
```

We also need to install some basic build tool for [node-gyp](https://github.com/nodejs/node-gyp) - node binaries build tool.
```
sudo apt-get install build-essential
```

I do not recommend on the "official" to install node, node rapid development life cycle (and the entire ecosystem for that matter) leads more then often for the need to have tight control over the node binaries version, or more likely to have to project running on different versions of node (such as [LTS](https://github.com/nodejs/Release) versions).

Before we can continue, we need `curl`:
```
sudo apt-get install curl
```

[NVM](https://github.com/creationix/nvm) tool gives you the ability to install and use multiple version of node, and prevent the (bad) usage of `sudo` for running node application, installation is via the command line:
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.5/install.sh | bash
```
Restart your terminal after install.

Now just install the latest the latest stable version of node:
```
nvm install stable
```
To update node just run the command again.

One thing you'll have to remember when use NVM is that you'll need to explicitly specify the node version you want to use (installing does it automatically on the end of the install), so next time you'll login to ubuntu you'll need to run the command:
```
nvm use stable
```

## Developing

Your windows hard disk drivers are mounted in linux under `/mnt/<drive letter>/` so you can access any folder via linux and run whatever command you need in order to get your project up and running, missing needed tools can be installed via `npm` or `apt-get` for system tools.

IDE terminal integration support is still on its early days, [vscode](https://code.visualstudio.com/) offers support with minimal configuration, [guide](https://code.visualstudio.com/docs/editor/integrated-terminal). If your IDE just point to the executable (like cmd.exe) check if you can point it to `c:\Windows\System32\bash.exe`.

In Visual Studio, hit settings in JSON format and add:
```json
{
    "terminal.integrated.shell.windows": "C:\\WINDOWS\\System32\\bash.exe"
}
```

# Check ports
I'm used to type `sudo netstat -lptn` to see if ports are already opened or if my server is running. You need to run
```bash
sudo apt-get install net-tools
```
... to install them

# MongoDB
Note: if you are using a different distro than Debian, look for the right key and list [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/)
```bash
sudo apt-get install dirmngr apt-transport-https
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
echo "deb http://repo.mongodb.org/apt/debian stretch/mongodb-org/4.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
sudo apt-get install -y mongodb-org
sudo mkdir -p data/db
```

To start your database, open up a new bash shell and type
```
sudo mongod --dbpath ~/data/db
```

Now in your original shell you can type
```bash
gius@my-awesome-machine-name:~$ mongo
MongoDB shell version v4.0.4
connecting to: mongodb://127.0.0.1:27017
/** blablabla **/
---

> show dbs;
admin   0.000GB
config  0.000GB
local   0.000GB
> exit
bye
```

# Sources

* [https://daverupert.com/2018/04/developing-on-windows-with-wsl-and-visual-studio-code/](https://daverupert.com/2018/04/developing-on-windows-with-wsl-and-visual-studio-code/) - post WSL settings for WebDev
* [http://www.akitaonrails.com/2017/09/20/windows-subsystem-for-linux-is-good-but-not-enough-yet](http://www.akitaonrails.com/2017/09/20/windows-subsystem-for-linux-is-good-but-not-enough-yet) - performance benchmark between Windows-host-VirtualBox-Linux-Guest, Bare-metal-Linux, Windows-native, Windows-WSL and Mac 
* [https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) 
* [https://github.com/Microsoft/WSL/issues/796](https://github.com/Microsoft/WSL/issues/796) issues with WSL and MongoDB and the solution: [https://gist.github.com/Mikeysax/cc86c30903727c556bcce960f7e4d59b](https://gist.github.com/Mikeysax/cc86c30903727c556bcce960f7e4d59b)
* Get to the version of WSL [https://github.com/Microsoft/WSL/issues/1728](https://github.com/Microsoft/WSL/issues/1728)

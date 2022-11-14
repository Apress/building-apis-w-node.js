# Setting up the environment

In this chapter, we are going to install Node.js on the most used OS (Windows, Linux and MacOSX). However, the entire book will use MacOSX as default OS.

Don't worry about the differences among these operation systems, because all examples in this book are compatible with the all the three OS platforms.

## Node.js standard installation

To set up the Node.js environment is quite the same despite the OS. Just few procedures will be different in each OS, especially for Linux, because in Linux environment you can compile an application or install it using a package manager, but it is not a big deal.

![Node.js homepage](images/nodejs-install.jpg)

The first step is to access the official Node.js website: [nodejs.org](http://nodejs.org)

Then, click on the button **v5.6.0 Stable** to download the latest and compatible version for your Windows or MacOSX. If you use Linux, I recommend you to read this quick wiki:

https://github.com/nodejs/node/wiki/Installing-and-Building-Node.js

This wiki explains how to compile and install Node.js in some popular Linux distro.

After the download, install it. If you use Windows or MacOSX, just click in some next, next and next buttons until the installation is completed, after all there is none specific configuration to do it.

To test if everything is running correctly, just open the **Terminal** (for Linux or MacOSX), **Command Prompt** or **Power Shell** (for Windows) type this command:

``` bash
node -v && npm -v
```

This will check the version of Node.js and NPM. This book uses **Node v5.6.0** and **NPM 3.6.0**.

> ##### About io.js and Node.js merge
> Since **September 8th 2015**, Node.js was upgraded from **0.12.x** to **4.0.0** version. This happened because of the merge from **io.js**. The io.js was a fork built and maintained by the Node.js community. They worked to include some features from **ECMAScript 6** implementation, besides making several improvements that were slowly on Node.js. The io.js reached the **3.0.0** version and both groups decided to merge io.js into Node.js, the result was the Node.js **v4.0.0**. This merge not only gave a huge upgrade to this platform, but also gave a more stable and reliable version to Node.js.

**Note:** This book will use the **Node v5.6.0**, with a lot of ES6 codes. To learn more about ECMAScript 6, take a look this website:
http://es6-features.org

## Node.js installation via NVM

**Warning:** This section, we are just going to explore an alternative way to setup the Node.js using a version manager. Feel free to skip this step in case you don't need or don't want to learn how to use the NVM.

Just like Ruby language has the RVM (*Ruby Version Manager*) to manage multiple Ruby versions, Node.js also has a manager too, the NVM (*Node Version Manager*).

NVM is a perfect solution to test your projects in different Node.js versions. It is also useful for people that like to test unstable or latest versions.

NVM has great advantages, it is practical, easy to use, it can uninstall a Node.js version with a single command and will save you time to search and install some Node.js version. It is a good alternative, especially on Linux which its native package managers are frequently outdated and invalidates a new Node.js version to be installed.

### Setup NVM

In few steps you can set up NVM into a MacOSX or Linux system, you just need to run this command:

``` bash
curl https://raw.githubusercontent.com/creationix/nvm/v0.30.2/install.sh | bash
```

Unfortunately, the official version of NVM is not available to Windows, but there are alternative projects created by the community. Here are two similar alternatives for Windows:

* **NVM-Windows:** https://github.com/coreybutler/nvm-windows
* **Nodist:** https://github.com/marcelklehr/nodist

Both have a similar CLI (*Command Line Interface*) inspired by NVM.

### Top NVM commands

As a cake recipe, right bellow I show you a list with the top commands that will be essential to manage multiple versions or at least keep your environment updated with the latest version.

* `nvm ls`: lists all versions installed;
* `nvm ls-remote`: lists all the available Node.js versions to download and install;
* `nvm install vX.X.X`: download and install a version;
* `nvm uninstall vX.X.X`: Uninstall an existing version;
* `nvm use vX.X.X`: choose an existing version to use;
* `nvm alias default vX.X.X`: choose an existing version to be loaded by default in the OS boot time;

**Attention:** change the `vX.X.X` for a Node.js version of your choice, such as `v5.6.0`

### Installing Node.js via NVM

To install Node.js via NVM, you just need to run the following commands:

``` bash
nvm install v5.6.0
nvm use v5.6.0
nvm alias default v5.6.0
```

After running these commands, you will have the Node.js installed and ready to use.

### Conclusion

Congrats! Now, besides having the Node.js installed and working, you also learned a new way to manage multiple Node.js versions via NVM. In the next chapter, we are going to explore some important things about NPM (*Node Package Manager*). So, keep reading, because the fun is just starting!

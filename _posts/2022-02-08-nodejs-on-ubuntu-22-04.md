---
layout: post
title: "How To Install Node.js on Ubuntu 22.04"
date: 2022-02-08
category:
  - Node.js
  - Ubuntu
image: assets/img/blog/ubuntu_nodejs.webp
author: Andy
tags: Ubuntu, JavaScript, Node.js
---

#### Introduction

[Node.js][nodejs-org] is a JavaScript runtime for server-side programming. It allows developers to create scalable backend functionality using JavaScript, a language many are already familiar with from browser-based web development.

In this guide, we will show you three different ways of getting Node.js installed on an Ubuntu 22.04 server:

 * using apt to install the nodejs package from Ubuntu’s default software repository
 * using apt with an alternate PPA software repository to install specific versions of the nodejs package
 * installing nvm, the Node Version Manager, and using it to install and manage multiple versions of Node.js

For many users, using `apt` with the default repo will be sufficient. If you need specific newer (or legacy) versions of Node, you should use the PPA repository. If you are actively developing Node applications and need to switch between node versions frequently, choose the `nvm` method.

#### Prerequisites

This guide assumes that you are using Ubuntu 22.04. Before you begin, you should have a **non-root** user account with sudo privileges set up on your system.

#### Option 1 — Installing Node.js with Apt from the Default Repositories

Ubuntu 22.04 contains a version of Node.js in its default repositories that can be used to provide a consistent experience across multiple systems. At the time of writing, the version in the repositories is 12.22.9. This will not be the latest version, but it should be stable and sufficient for quick experimentation with the language.

> **Warning:** The version of Node.js included with Ubuntu 22.04, version 12.22.9, is an LTS, or “long-term support” release. It is technically outdated, but should be supported until the release of Ubuntu 24.04.

To get this version, you can use the `apt` package manager. Refresh your local package index first by typing:
```bash
  $ sudo apt update
```
Then install Node.js:
```bash
  $ sudo apt install nodejs
```
Press `Y` when prompted to confirm installation. If you are prompted to restart any services, press ENTER to accept the defaults and continue. Check that the install was successful by querying node for its version number:
```bash
  $ node -v
```
Output
```  
  v12.22.9
```
If the package in the repositories suits your needs, this is all you need to do to get set up with Node.js. In most cases, you’ll also want to also install `npm`, the Node.js package manager. You can do this by installing the `npm` package with `apt`:
```bash
  $ sudo apt install npm
```
This will allow you to install modules and packages to use with Node.js.

At this point you have successfully installed Node.js and npm using apt and the default Ubuntu software repositories. The next section will show how to use an alternate repository to install different versions of Node.js.

#### Option 2 — Installing Node.js with Apt Using a NodeSource PPA

To install a different version of Node.js, you can use a PPA (personal package archive) maintained by NodeSource. These PPAs have more versions of Node.js available than the official Ubuntu repositories. Node.js v14, v16, and v17 are available as of the time of writing.

First, we will install the PPA in order to get access to its packages. From your home directory, use curl to retrieve the installation script for your preferred version, making sure to replace **17.x** with your preferred version string (if different).

```bash
  $ cd ~
  $ curl -sL https://deb.nodesource.com/setup_17.x -o nodesource_setup.sh
```
Refer to the NodeSource documentation for more information on the available versions.

You can inspect the contents of the downloaded script with `nano` (or your preferred text editor):
```bash
  $ nano nodesource_setup.sh
```
Running third party shell scripts is not always considered a best practice, but in this case, NodeSource implements their own logic in order to ensure the correct commands are being passed to your package manager based on distro and version requirements. If you are satisfied that the script is safe to run, exit your editor, then run the script with `sudo`:
```bash
  & sudo bash nodesource_setup.sh
```
The PPA will be added to your configuration and your local package cache will be updated automatically. You can now install the Node.js package in the same way you did in the previous section. It may be a good idea to fully remove your older Node.js packages before installing the new version, by using `sudo apt remove nodejs npm`. This will not affect your configurations at all, only the installed versions. Third party PPAs don’t always package their software in a way that works as a direct upgrade over stock packages, and if you have trouble, you can always try to revert to a clean slate.
```bash
  $ sudo apt install nodejs
```
Verify that you’ve installed the new version by running `node` with the `-v` version flag:
```bash
  $ node -v
```
Output
```  
  v17.6.0
```
The NodeSource `nodejs` package contains both the `node` binary and `npm`, so you don’t need to install `npm` separately.

At this point you have successfully installed Node.js and `npm` using `apt` and the NodeSource PPA. The next section will show how to use the Node Version Manager to install and manage multiple versions of Node.js.

### Option 3 — Installing Node Using the Node Version Manager

Another way of installing Node.js that is particularly flexible is to use nvm, the Node Version Manager. This piece of software allows you to install and maintain many different independent versions of Node.js, and their associated Node packages, at the same time.

To install NVM on your Ubuntu 22.04 machine, visit the project’s GitHub page. Copy the `curl` command from the README file that displays on the main page. This will get you the most recent version of the installation script.

Before piping the command through to bash, it is always a good idea to audit the script to make sure it isn’t doing anything you don’t agree with. You can do that by removing the `| bash` segment at the end of the `curl` command:
```bash
  $ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh
```
Take a look and make sure you are comfortable with the changes it is making. When you are satisfied, run the command again with `| bash` appended at the end. The URL you use will change depending on the latest version of nvm, but as of right now, the script can be downloaded and executed by typing:
```bash
  $ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```
This will install the `nvm` script to your user account. To use it, you must first source your `.bashrc` file:
```bash
  $ source ~/.bashrc
```
Now, you can ask NVM which versions of Node are available:
```bash
  $ nvm list-remote
```
Output
```  
  . . .
       v16.11.1
       v16.12.0
       v16.13.0   (LTS: Gallium)
       v16.13.1   (LTS: Gallium)
       v16.13.2   (LTS: Gallium)
       v16.14.0   (Latest LTS: Gallium)
        v17.0.0
        v17.0.1
        v17.1.0
        v17.2.0
        v17.3.0
        v17.3.1
        v17.4.0
        v17.5.0
        v17.6.0
```
It’s a very long list! You can install a version of Node by typing any of the release versions you see. For instance, to get version v16.14.0 (another LTS release), you can type:
```bash
  $ nvm install v16.14.0
```
You can see the different versions you have installed by typing:
```bash
  nvm list
```
Output
```  
->     v16.14.0
default -> v16.14.0
iojs -> N/A (default)
unstable -> N/A (default)
node -> stable (-> v16.14.0) (default)
stable -> 16.14 (-> v16.14.0) (default)
lts/* -> lts/gallium (-> v16.14.0)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.24.1 (-> N/A)
lts/erbium -> v12.22.10 (-> N/A)
lts/fermium -> v14.19.0 (-> N/A)
lts/gallium -> v16.14.0
```
This shows the currently active version on the first line (`-> v16.14.0`), followed by some named aliases and the versions that those aliases point to.
> **Note:** if you also have a version of Node.js installed through `apt`, you may see a `system` entry here. You can always activate the system-installed version of Node using `nvm use system`.

We can install a release based on these aliases as well. For instance, to install `fermium`, run the following:
```bash
  $ nvm install lts/fermium
```
Output
```  
Downloading and installing node v14.19.0...
Downloading https://nodejs.org/dist/v14.19.0/node-v14.19.0-linux-x64.tar.xz...
################################################################################# 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v14.19.0 (npm v6.14.16)
```
You can verify that the install was successful using the same technique from the other sections, by typing:
```bash
  $ node -v
```
Output
```  
v14.19.0
```
The correct version of Node is installed on our machine as we expected. A compatible version of `npm` is also available.

#### Conclusion
There are a quite a few ways to get up and running with Node.js on your Ubuntu 22.04 server. Your circumstances will dictate which of the above methods is best for your needs. While using the packaged version in Ubuntu’s repository is the easiest method, using `nvm` or a NodeSource PPA offers additional flexibility.

[nodejs-org]: https://nodejs.org/
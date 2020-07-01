+++
author = "James D Hughes"
categories = ["nodejs", "tutorial", "ubuntu", "npm"]
date = 2015-04-08T03:42:01Z
description = ""
draft = false
slug = "installing-node-without-sudo"
tags = ["nodejs", "tutorial", "ubuntu", "npm"]
title = "Installing Node without Sudo - Ubuntu 14.04 LTS"

+++


If you're unlike me and really don't care about superuser permissions or having the most up-to-date releases, you can just do the regular old naive thing and use the package manager...
```
sudo apt-get update && sudo apt-get install nodejs
```
However since you're here, I'm betting this doesn't jive with you.

The idea of having to give checked out source code from the npm repository super user access makes me quite uncomfortable. I mean, all it's going to do is sit on a virtual machine that will remain dormant long after I complete an awesome a-snychronous ToDo application and feel like a boss. It's the principal of the matter!

Let's get down to it then.
This installation method only works for the user it was installed for in their home directory.
Let's start by installing curl
```
sudo apt-get install curl
```
Just for cleanliness, let's make a new folder for us to download the node distribution to.
```
mkdir ~/node-latest
cd ~/node-latest
```
Next we are going to want to download Node.js using super awesome CLI's
```
curl http://nodejs.org/dist/node-latest.tar.gz
```
Let's then extract the tarball using every option available.
```
tar -xvzf node-latest.tar.gz
```
Here is where we are going to want to configure where we want to install Node.
Let's make a directory in our home to hold the user-only access to the binaries.
```
mkdir ~/local
```
Still in the ~/node-latest folder, execute the following commmand replacing the /PATH/TO/LOCAL be equivalent to your /home/$USER/local. replace 'james' with your \*nix username
```
./configure --prefix=/home/james/local
```
Note that I didn't use the ~ do find my home. This is by trial and error.  Maybe it's the compiler that I was using. Maybe it's the distribution that I'm using (Ubuntu 14.04 LTS), or maybe I'm just amazingly unintelligent. The fact of the matter is when I used the magical tilde, it made a symlink in the ~/node-latest that linked back to my ~ and I have no idea where the binaries were actually installed. That is why I have used a hard reference to my user and only my user for the binaries.

From here, let's go ahead and compile the libraries

```
make install
```
Grab a coffee, soda, beer, whatever your vice is while you wait for this to complete.
We're going to want to append the following to your ~/.bashrc file to add the ~/local/bin to your path.

```
vi ~/.bashrc
```
Scoll to the bottom and add
```
export PATH=$HOME/local/bin:$PATH
```
This can also be solved by piping to the file with 
```
echo 'export PATH=$HOME/local/bin:$PATH' >> ~/.bashrc
```
Then you want to refresh your profile
```
. ~/.bashrc
```

That's it!
try the following commands to make sure that your cli tools are working:
```
node -v
```
```
npm -v
```
If you get some version numbed spit out then you're all good. Else some black magic occurred and you might want to start over. 


Happy coding!


---
layout: page
title: Dev Environment Setup
---

Before we can do much programming, we need to make sure our machines are properly configured with a functional development environment. Let's walk through this process now to make sure we have what we need.

Here are the basics we're going to go over:

*   A text editor. Developers need to work with text in a different way than
your average MS Word user. We'll want a text editor designed for software
development.
*   A system "Terminal" for interacting with our machine from the command
line. Fortunately OS X already ships with one.
*   OS X "Command Line Tools" -- these are some system dependencies needed for
some of the tools we will use.
*   HomeBrew -- This is a "package manager" for installing other developer-related
programs. You can think of it as the "App Store for nerds."
*   Git (An application for handling "version control" of our software projects)
*   The Ruby programming language, as well as a Ruby "Version Manager" to allow
us to install other versions as needed

### Text Editor

If you don't already have a favorite text editor, we recommend using [Atom](https://atom.io/).

Atom is a program where we edit code - it is a text editor with many great features that makes editing code more enjoyable compared to a simple text editor. Atom is commonly used in the software development industry, and we use it throughout your time at Turing.

### Browser

Similarly, if you don't already have a favorite browser, we recommend using [Google Chrome](https://www.google.com/chrome/). Google Chrome is a web browser that is great for general internet perusing, but web developers like it because it has very useful developer tools (the console), which we will touch on in the prework and use extensively during class.

### Terminal

The terminal is a textual interface to your computer. Before Graphical User Interfaces were invented, this was the only way one could interact with a computer.

Nowadays, developers still prefer this interface due to its power, flexibility, and speed.

A Terminal allows you to navigate around folders (called directories) and run programs. For example, when we run `ruby`, we are running that program from the terminal.

To launch the terminal, open Spotlight using `Command-Spacebar`, type "terminal", then enter.

If this is all new for you, see [Terminal and Editor](./terminal_and_editor)

#### Setting Up Terminal Access for Atom

One of the things you'll do frequently is open an entire folder (like when working on a project) in your text editor. Let's get that setup:

Open up Atom and in the menu bar, drop down the `Atom` menu. Within that, click on `Install Shell Commands` and now atom should be enable from your command line.

To confirm that atom is working from your command line, enter `atom .` in your terminal. If it is setup correctly, the atom editor will automatically open. If it does not open atom and an error occurs instead, try entering this in the command line: `ln -s /Applications/Atom.app/Contents/Resources/app/atom.sh /usr/local/bin/atom` and try the first command (`atom .`) again.

Atom also offers a number of different options and packages that you can customize to your liking. [This](https://www.youtube.com/watch?v=WWwBQQOGllo&list=PLYzJdSdNWNqwNWlxz7bvu-lOYR0CFWQ4I) series of videos will walk you through many of them if you'd like to dive deeper.

### XCode & Command Line Tools

XCode is a huge suite of development tools published by Apple. If we wanted to develop software for the Apple Ecosystem (iPhone apps, Mac OS Apps, etc), we would use XCode as our editor. But even if we aren't working in this ecosystem, XCode provides some system dependencies that we'll want to have available.

You'll want to install it before attempting to install anything else.

1.  Install XCode from the Apple App Store (this will probably take a little while to finish)
2.  Open the application after installing and agree to the SLA terms
3.  Open `terminal` and run `xcode-select --install`, enter your user password

Now you should have the underlying tools we need to move forward.

### Homebrew

[Homebrew](http://brew.sh) is a package management system that makes it easy to install hundreds of open source projects and compile them from source for maximum performance on your machine.

Open the Terminal then run the homebrew installation script:

```shell
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

It will ask you for your password. This is the password to log in to your account on the computer.
It needs this because it installs its packages in a place that all users of this computer can access.

#### Verifying Homebrew

When it has completed the installation run `brew doctor` and it should tell you that everything is fine:

```shell
brew doctor
Your system is ready to brew.
```

#### Modifying your PATH

If you got a warning from Homebrew about your path, do the following:

```shell
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```

Now run `brew doctor` again and the warning should be gone.

__Aside: `PATH`:__

Your `PATH` is a system configuration property which tells your computer which places to look for underlying programs when you want to run a command.

For example, when we type `ruby` at the command line to run a ruby program, our `PATH` will help the system know where on the system to find ruby. By adding this directory to our `PATH`, we're telling the system how to find the various applications we will install using Homebrew.

__Aside: `~/.bash_profile`__

When we use our terminal, we're actually using a program called a "Shell" to interact with the underlying Operating System. Specifically, we're using a shell called [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)).

The file `~/.bash_profile` contains settings and commands to help us configure the shell, so when we have a bit of configuration code such as setting our `PATH`, it often goes in our `~/.bash_profile`.

### Git

[Git](http://git-scm.com/) is the version control system of choice in the Ruby community. XCode installed an older version of Git for you, but let's update it.

```shell
brew install git
==> Downloading http://git-core.googlecode.com/files/git-1.8.3.4.tar.gz
########################################################### 100.0%
```

#### Configuring Git

If you haven't used git before (don't worry, we'll be covering its usage in future classes), we'll want to configure it with some basic information about us.

We can tell git to configure itself using the `git config` command from our terminal. Additionally, we're setting "global" configurations for git, so we'll use the `--global` flag when we provide it with a new piece of configuration.

Tell git your Name and Email address by using the following commands, substituting your own name and email:

```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

### [rbenv](https://github.com/rbenv/rbenv#homebrew-on-mac-os-x)

As the Ruby language has evolved over the years, new versions have been released containing new features and various upgrades. The first version, released in 1995, was 0.95, and as of this writing we're at 2.4.

To some extent programs written for one version of Ruby will run just fine on another version,
but sometimes you'll encounter incompatibilities, such that a program needs to be run with a specific version of Ruby.

For this reason, we'd like to be able to install and manage multiple versions of Ruby on our system. This is precisely the job rbenv handles.

#### Installation

Similar to Homebrew, rbenv provides a script to get everything installed. Run this in your Terminal:

```shell
brew update
```

```shell
brew install rbenv
```

```shell
rbenv init
```

The output from your terminal will be something similar to:

```shell
# Load rbenv automatically by appending
# the following to ~/.bash_profile:

eval "$(rbenv init -)"
```

This means that you will need to add the above line (beginning with `eval`) to the bottom of your bash profile.

In your terminal, type:

```shell
atom ~/.bash_profile
```

This will open up your bash_profile in Atom so you can edit it. Copy the line `eval "$(rbenv init -)"` and paste it at the END of your bash_profile. Save the file.

Check if you did this step correctly by switching back to your terminal and typing `cat ~/.bash_profile`. You should see `eval "$(rbenv init -)"` somewhere in the output of your terminal.

Close your terminal and reopen it. This is a very important step since the bash profile is loaded each time a new terminal window is opened.

Now check to make sure rbenv was installed properly. In your terminal, type:

```
rbenv versions
```

It should give you a version number rather than an error message.

More information about rbenv can be found [here](https://github.com/rbenv/rbenv#homebrew-on-mac-os-x)

### Ruby

Now that we have rbenv installed, we're going to use it to install a specific version of Ruby: Ruby 2.4.1

If you need another version it'll be same procedure, just replace "2.4.1" in the instructions with whichever version you want.

Install it with:

```shell
rbenv install 2.4.1
```

It should take a while to finish installing. Type

```shell
rbenv versions
```

and you should now see 2.4.1 listed.

Be careful, there are two different rbenv commands, `version` and `versions`. The first shows you your current version. The second shows all installed versions.

Switch to your newly installed version with

 `rbenv local 2.4.1`

 Now enter:

 `ruby -v`

 This shows us what version of Ruby we are running. You should see something like:

 `ruby 2.4.1p205 (2017-12-14 revision 61247) [x86_64-darwin17]`

 You can ignore everything after the "p". This output shows us we are running Ruby 2.4.1, which is what we want. If you got something different than 2.4.1, go back through the Rbenv installation, make sure you have you successfully edited your bash_profile, restart your terminal, and try again.

#### Setting the Default Version

You can tell rbenv which Ruby version you want to use by default:

```shell
rbenv global 2.4.1
```

To reload your shell, do the following:

```shell
rbenv rehash
```

### Folder Structure

Now, let's create a folder structure to store all of your code. We can start by moving to our home folder, and then creating a Turing folder.

```
$ cd ~
$ mkdir turing
```

__A few notes:__

*   `cd` stands for "change directory", and moves us to a specific place on the filesystem, similar to using the GUI Finder to browse directories on the machine
*   `~` is a shortcut for our "home directory". It will be in a place like `/Users/<your-user-name>`
*   `mkdir` stands for "make directory", and it allows us to create new folders on the machine

At this point, we want to enter the directory we have created, and then we will create ourselves a directory for the first module. We call this new directory 1module, so we can use tab complete more easily.

```
cd turing
mkdir 1module
```

Now that this is complete, you can now get to your first module folder from anywhere through the terminal by typing:

```
cd ~/turing/1module
```

__Try It:__ Move to your home directory using `cd ~`. Then use `cd` to navigate back to your `1module` directory.

Once we get exposed to more projects, homework, etc, our ultimate 1module folder setup will look like this:

```
1module/  (no git)
    - homework/ (git)
    - classroom_exercises/ (git)
    - morning_exercises/ (git)
    - projects/ (no git)
        - project_name_1/ (git)
        - project_name_2/ (git)
        - project_name_3/ (git)
        - etc.
```

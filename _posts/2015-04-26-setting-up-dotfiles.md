---
layout: post
title: Setting up a personal dotfiles repository
category: "dev"
comments: true
---

The configuration files in your home directory are called `dotfiles` because of their preceding dot in the file name which makes them hidden. Good examples are

* `.gitconfig`
* `.zshrc`
* `.vimrc`

Having your dotfiles under version control is useful for many reasons. Be it for setting up a new machine with ease, having the history of changes to the individual files or – given the repository is publicly accessible – helping others with their configuration by showing them yours.

To accomplish that, you could simply do a `git init` inside your home folder. But then you would have to ignore all the files and folders you never want to have in a repository. Imagine folders like `.ssh` or similarly sensitive data to be stored on a remote server. Urgs!

## Creating the repository

Instead of blacklisting unwanted files, we are whitelisting selectively. We do that by creating a dotfiles subfolder in our user home, moving the config files into it and symlink them.

{% highlight bash %}
$ mkdir ~/dotfiles
$ mv ~/.gitconfig ~/dotfiles/gitconfig
$ ln -s ~/dotfiles/gitconfig ~/.gitconfig
{% endhighlight %}

Please note the missing dot in the new filename. I think it's more convenient to have the files visible by default.

Afterwards, all we have to do is creating a new git repository.

{% highlight bash %}
$ cd ~/dotfiles
$ git init
{% endhighlight %}

## Installation on a new machine

When setting up a new computer you first clone the repository.

{% highlight bash %}
$ cd ~
$ git clone https://github.com/[user]/dotfiles
{% endhighlight %}

For symlinking the files, i wrote a small script which also backups existing dotfiles to a folder.

{% gist 509b9bfc643f40c099f9 %}

After cloning, you simply execute the script with `./install` and everything should be working. Feel free to use and modify this script in your own repository.

Now go and do the same with your dotfiles! If you are interested, you can [check out my dotfiles repository on GitHub](https://github.com/hpurmann/dotfiles).

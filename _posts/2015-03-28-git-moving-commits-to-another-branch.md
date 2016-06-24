---
layout: post
title: Moving commits to another branch
category: "dev"
comments: true
---

Let's say you made a few commits and then realized that you did them to the wrong branch. Because git is awesome, it's really easy to change the branch you committed to.

Let's learn why!

## Commits

Every commit stores the SHA-1 hash of its parent commit(s). Think of it as a directed acyclic graph with each node pointing to its parent(s).

![Simple Commits](/assets/graphimages/commit-1.png)

## Branches

A branch is just a pointer to a commit. Git only stores a single file with the filename being the name of the branch. Inside, there is only the SHA-1 of the top-most commit.

![Simple branching](/assets/graphimages/commit-2.png)

Or to quote the great book [Git Internals](https://github.com/pluralsight/git-internals-pdf):

> Creating a branch is nothing more than just writing 40 characters to a file.

Go for it and have a look in the `.git` directory of some git repository.

{% highlight bash %}
$ cd .git
$ ls
COMMIT_EDITMSG config         hooks          info           objects
HEAD           description    index          logs           refs
$ cd refs/heads
$ ls
master
{% endhighlight %}

The folder `.git/refs/heads` stores the branch reference files.

{% highlight bash %}
$ cat master
eb5c3831d6ebca824857d30cea70948201529ada
{% endhighlight %}

Let's say you are on the master branch and create another branch named `dev`.

{% highlight bash %}
$ git branch dev
$ cat dev
eb5c3831d6ebca824857d30cea70948201529ada
{% endhighlight %}

The new branch `dev` is pointing to the same commit as master. If we draw that, it would look like this:

![New branch dev](/assets/graphimages/commit-3.png)

## Back to the problem ...

With the knowledge of commits and branches you can probably come up with a solution yourself.

![Commits on wrong branch](/assets/graphimages/commit-4.png)

At this point you noticed that you want commits `D` and `E` on a new branch called `feature1`.

{% highlight bash %}
$ git branch feature1
{% endhighlight %}

![Create a feature branch](/assets/graphimages/commit-5.png)

To "remove" the commits from the master branch, you simply move the branch pointer two commits back. Please note the two carets (`^`) behind `HEAD`.

{% highlight bash %}
$ git reset --hard HEAD^^
{% endhighlight %}

![Move branch pointer back](/assets/graphimages/commit-6.png)

Because commits are just referencing their parents, `D` and `E` are now unreachable from `master`. Now you can just switch to your new branch and keep working on it.

{% highlight bash %}
$ git checkout feature1
{% endhighlight %}

![Checkout feature1 branch](/assets/graphimages/commit-7.png)

I hope you agree with me that this is really easy once you understood what these commands do internally. For further reading, I highly recommend reading the [Think-like-a-git](http://think-like-a-git.net/) website and the mentioned [Git Internals](https://github.com/pluralsight/git-internals-pdf) book by Peepcode.

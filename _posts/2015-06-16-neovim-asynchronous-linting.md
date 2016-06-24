---
layout: post
title: Fast, non-blocking plugin execution with neovim
category: "dev"
comments: true
---

Recently, I've been working with a bunch of rather big CoffeeScript files. Since I have [coffeelint](http://www.coffeelint.org/) installed, [syntastic](https://github.com/scrooloose/syntastic) is checking the current buffer for possible linting errors on every save. This is slow. *Really* slow. It takes about one second for a 1000 line file.

In that time, vim is blocked. Even worse, when trying to change the buffer while waiting, the UI gets completely messed up by mixing the buffers contents.

Last weekend, I stumbled across [Neomake](https://github.com/benekastah/neomake) which is implementing asynchronous`:make` using the [new job control API](http://neovim.io/doc/user/job_control.html) provided by [neovim](http://neovim.io/).

That looked promising. Finally I had a good reason to replace vim by neovim. I had been following the progress on neovim for a couple of months.

Switching in most cases just means renaming your `vimrc` to `nvimrc`. I took a different approach. Instead of just renaming, I started from an empty config, *copying over one by one*. I really do recommend that because that way I could get rid of a few commands which are either [just default in neovim](https://github.com/neovim/neovim/wiki/Differences-from-vim) or which were hacks to make vim work in a sane way.

If you want your cursor to be narrow in insert mode and wide in normal mode, the [configuration is a bit different than in vim](https://github.com/neovim/neovim/wiki/FAQ#how-can-i-change-the-cursor-shape-in-the-terminal).

## Neomake config

After installing neomake, simply specify which maker you want to use for which file type. For instance, `coffeelint` is enabled with

{% highlight vim %}
let g:neomake_coffeescript_enabled_makers = ['coffeelint']
{% endhighlight %}

It makes sense to run a linter on every every buffer save. This auto command will do the job.

{% highlight vim %}
autocmd! BufWritePost * Neomake
{% endhighlight %}

## Comparison

Let's see how that looks compared to vim. First, vim with syntastic:

<script type="text/javascript" src="https://asciinema.org/a/22055.js" id="asciicast-22055" async></script>

And this is neovim with Neomake:

<script type="text/javascript" src="https://asciinema.org/a/22048.js" id="asciicast-22048" async></script>

As you can see, the UI is not blocked after performing the save. Instead, the results from coffeelint are being applied asynchronously as soon as they are available.

## The future is here

Indeed, the [job control implementation](https://github.com/neovim/neovim/pull/475) is a killer feature of neovim. Thiago's rejected PR to vim was the reason for creating the fork in the first place. They also implemented a [terminal emulator](https://github.com/neovim/neovim/pull/2076) inside of neovim.

But there are lots of other important things in which the neovim collaborators are making [great progress](https://github.com/neovim/neovim/wiki/Progress). For instance, they introduced continuous integration with code coverage, switched to CMake as the build system and automated the generation of documentation, analysis and nightly builds. The reponse to [pull requests](https://github.com/neovim/neovim/wiki/Contributing) is mostly quick and positive.

All of these are signs of a modern software development environment. Additionally, features being worked on sound very promising. For instance, they want to extract away the GUI from the core C codebase. That will allow programs to use neovim under the hood. Imagine your mail program or your browser working with vim key bindings *without re-implementing vim behaviour*. One project which showcases this is the [vim-mode for the Atom editor](https://github.com/carlosdcastillo/vim-mode).

Still not convinced? Then read the [very nice neovim write-up by Geoff Greeg](http://geoff.greer.fm/2015/01/15/why-neovim-is-better-than-vim/), the creator of [the silver searcher](https://github.com/ggreer/the_silver_searcher.).

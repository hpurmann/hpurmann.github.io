---
layout: post
title: Show me what you got – in your terminal
category: "dev"
comments: true
---

<script type="text/javascript" src="https://asciinema.org/a/22634.js" id="asciicast-22634" async data-autoplay="true" data-speed="1.5"></script>

Ever wanted to show someone your terminal output?

Lately I've found myself wanting that. As I was writing the [speed comparison between vim and neovim with corresponding plugins](http://hpurmann.com/2015/06/16/neovim-asynchronous-linting/) it was clear to me that a visual representation would be really helpful.

So I chose the obvious solution – animated gifs. Sadly, there is always the trade-off between small size and high quality and I needed several attempts to record them.

After the post was online I found [asciinema](https://asciinema.org/) – and it blew my mind. It's a program which records everything happening in a terminal instance, including colors. Asciinema uses a pseudo-terminal which captures input and output together with the time elapsed between them and saves that to a file.

Asciinema also offers a free platform to upload and view recorded sessions. You can even embed a JavaScript web player as you've seen above.

Imagine someone having a problem with a tool not working as expected. He opens an issue on GitHub, explaining his problem. Normally he would attach a log file. But with asciinema, he is able to show everything he did and all the system's responses.

## Recording an opened program

For the post about neovim, I wanted to exclude the startup of the editor and navigation to the right file from the actual recording. I wanted it to be as short and concise as possible.

The asciinema recorder has an option to pass in a command. It is not part of the recording and asciinema automatically stops after it's execution. This was exactly what I needed. I prepared everything inside of a tmux session named `vim`, then detached and called asciinema like this:

{% highlight bash %}
asciinema rec -c "tmux attach -t vim"
{% endhighlight %}

To get rid of the tmux status line, I disabled it temporarily.

{% highlight bash %}
set status off
{% endhighlight %}

With this, I could replace the blurry gifs with lightweight, purely text based terminal recordings. And you can do the same.

I would love to hear about your personal use case.

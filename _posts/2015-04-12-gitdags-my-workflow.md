---
layout: post
title: My workflow with gitdags
category: "dev"
comments: true
---

As I [wrote in my last post]({% post_url 2015-04-05-gitdags %}), I like to
use the LaTeX package `gitdags` when a visual representation of an exemplary git
graph is needed.

In this post, I shortly show my workflow when creating these graphs.

{% highlight tex %}
\documentclass[preview]{standalone}

\usepackage{gitdags}

\begin{document}
    \centering
    \begin{tikzpicture}
      % Commit DAG
      \gitDAG[grow right sep = 2em]{
        A -- B -- C
      };
    \end{tikzpicture}

\end{document}
{% endhighlight %}

produces this graph

![Simple Commits](/assets/graphimages/commit-1.png)

The import thing to note here is the chosen `documentclass` which crops the
resulting pdf to be as big as the content.

{% highlight tex %}
\documentclass[preview]{standalone}
{% endhighlight %}

When running `pdflatex graph.tex` you would get such a pdf. To convert the pdf
into a png, I use [ImageMagick](http://www.imagemagick.org/). On a Mac, it's as
easy to get as to type

{% highlight bash %}
$ brew install imagemagick
{% endhighlight %}

Then, for converting the pdf into png, use ImageMagicks `convert` method.

{% highlight bash %}
$ convert graph.pdf graph.png
{% endhighlight %}

I wrote a small bash script which lets me specify input and ouput directories,
does the conversion from tex to png and later throws away all pdf generation
files.

{% gist 31c4d4b5c2c5acfa31b3 %}

A Makefile would probably be much better as it would only build files which
changed, not the whole directory. As for now, I will stick with the bash script
given that the amount of files is rather small. If I ever switch to Make, I'll update
this post accordingly.

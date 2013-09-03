---
title: An efficient setup to produce presentations
author: Mog
---

Recently I had to write several presentations and I wasn't happy with
the tools I was used to ([Beamer](https://en.wikipedia.org/wiki/Beamer_%28LaTeX%29) + [AUCTeX](https://www.gnu.org/software/auctex/)).

I read a little bit about what my friends were currently using and
gave a try to [several HTML5/CSS3/js solutions](http://www.bortzmeyer.org/logiciel-presentation-nouveau.html). The one that impressed
me the most was [DZSlides](http://paulrouget.com/dzslides/) by Paul
Rouget, especially for its awesome control panel (called shell in the
DZSlides doc). Even the basic `onstage.html` one, provided with the
vanilla framework, gives you lots of tools to work with: a preview of
the next slide, a timer and notes hidden from the slides to help you
out during the presentation.

The basics of DZSlides are simple: you complete an HTML template with
your slides (specified with regular HTML5 and CSS3), and use a shell
(another html file), to launch a presentation. For example, if your
presentation file is `slides.html` and if you want to use the
`onstage.html` shell, you'd have to browse to the address
`onstage.html#slides.html`.

The main problem with that particular framework from my perspective
was quite embarassing: it required a good habit of producing HTML5 and
CSS3 to be productive, and those are two technologies I barely
touched recently.

Here enters
[Pandoc](http://johnmacfarlane.net/pandoc/demo/example9/producing-slide-shows-with-pandoc.html)
and its ability to turn Markdown into several presentation frameworks
outputs. The page linked provides a quick and sufficient introduction
on the specifics of Pandoc's Markdown conversion semantics. Here is a
sample command to compile a markdown file into a DZSlides
presentation:

```shell
pandoc --smart --standalone \
       --from markdown_github+pandoc_title_block \
       --to dzslides \
       input.md \
       --output.html
```

However, the default styling isn't very good. Something you can do is
retrieve the CSS from one of Paul Rouget's sample presentations and
integrate it in the template used by Pandoc, or roll one of your
own. To get the current template used by pandoc, use:

```shell
pandoc -D dzslides > template.html
```

Then it is just a matter of finding the appropriate `style` section
and replacing it with a fancier one. To use the new template during
compilation, we just need to add an argument:

```shell
pandoc --smart --standalone \
       --from markdown_github+pandoc_title_block \
       --to dzslides \
       --template template.html \
       input.md \
       --output.html
```

Now that we have a nice Markdown to slideshow converter, let's look
more in depth at the workflow in use. Currently we need to edit a
Markdown file, compile an html presentation from it using Pandoc and
refresh a webpage to see the result of our modifications. That's three
steps. A quick fix to the need of html compilation is to use
[inotify-tools](https://github.com/rvoicilas/inotify-tools/wiki) in
order to know when the file changes to compile it automatically. Then,
to avoid refreshing the shell webpage, we can use an add-on such as
[AutoReload](https://addons.mozilla.org/en-us/firefox/addon/auto-reload/)
to save another action. To ease the combination of those tools, we can
use a Makefile as follows:

```shell
markdown = $(shell ls *.md)
html = $(markdown:.md=.html)
onstaged = $(html:%=onstage.html\#%)

all: $(html)

%.html: %.md
	pandoc \
		--smart --standalone \
		--from markdown_github+pandoc_title_block \
		--to dzslides \
		--template template.html \
		$< \
		--output $@
	touch onstage.html

view: $(html)
	firefox $(onstaged)
	while :; do inotifywait -qqe modify $(markdown); make; done

.PHONY: all view
```

The `touch` after the compilation is needed to say to the web browser
that our shell needs to be reloaded.

With this Makefile and the refresher add-on combined, the workflow is
straightforward: at the beginning of a session, fire up a terminal and
do a

    make view

in the presentation directory.

Afterwards, the presentation page is refreshed after each modification
and is written in an intuitive and efficient language,
e.g. Markdown. Enjoy! The only thing that I'm bothered with at this
point is that it loses navigation info, e.g. at which page of the
presentation you were before the refresh. Any idea on how to handle
that will be appreciated.
+++
date = "2017-02-05T03:05:56-05:00"
title = "The markup language known as Markdown"
Categories = ["blogging", "Hugo"]
Tags = ["Markdown","blogging","software","Hugo","Typora"]
Description = "What it is and how to use it"
+++


## What it Is

[Markdown](https://en.wikipedia.org/wiki/Markdown) is a *lightweight markup language* (as opposed to a **heavyweight** one like HTML or LaTeX). If you've ever taken plain-text file notes and used an asterisk to represent a bullet point, or a line of dashes like an underline for a heading, then you've basically already written Markdown. Markdown is a natural-looking "syntax" that lets you turn text like this:

```
## What it is
[Markdown](https://en.wikipedia.org/wiki/Markdown) is a *lightweight markup language* (as opposed to a **heavyweight** one like HTML or LaTeX).
```

into the HTML+CSS page that you're looking at right now. We've all known that annoying nerd who refuses to send HTML-formatted emails and insists on sending plain text peppered with slashes and asterisks instead of italics and bold? Yea, basically a guy like that turned it into a semi-official standard, and now instead of being imaginary italics and bold, it actually renders that way.

## Why Not Just Write in HTML Directly?

I think this is [best explained by Brett Terpstra here](http://brettterpstra.com/2011/08/31/why-markdown-a-two-minute-explanation/). Basically, HTML sucks to work in, if you're just trying to write some content. HTML is tedious, hard on the eyes, and error prone. If you want a site to look good at all, it also needs CSS, and that sucks even worse. Real people don't want to work in either one, they just want to write prose and have it look good. For this, you can use Markdown as a standard format for your thoughts, and then let a static site generator (like Hugo) turn it into a web page. Be a writer, not a website developer, is the thinking.


## Editors

I went looking for the ideal text editor in which to edit Markdown files. Ideally I could find something that ran on both MacOS and Linux, just for consistency since I use both. But Markdown is a standard format so I would also settle for the best on each respective platform, even if I had to use two different editors.

If there is one thing that gets reinvented most often, it's text editors. Programmers love to re-solve their own nerd problems rather than tackle real-world problems, and one problem every programmer has is editing text. So there are about 500 choices of text editor at this point, but I narrowed it down to these 7 just on the basis of using them to edit Markdown.

![MarkdownEditors](img/markdownEditors.png)

If the job is programming, a text editor should have code-completion and syntax highlighting. But if the job is editing a markup language, WYSIWYG is the number one thing I care about. So you notice I am not even considering the Vims and Emacs of the world. The whole point of Markdown was to be easy: easy for a human to read in its raw state, easy to edit. But I guess I would take this sentiment a little bit further: why should I have to look at markup language at all? It's 2017, shouldn't I have an editor at least as good as WordPad from Windows 95? Of course, I want it to be able to flip back to the raw "source" markup when necessary, but most of the time I just want to edit it the way it's going to look when I'm done.

This is a surprisingly uncommon feature. It seems that most editors have adopted the two-view or split-window paradigm seen in editors for more complicated markup and typesetting languages like LaTeX. They present the raw Markdown source on the left, and the rendered version on the right. Booooo. The concept of Auto-save, on the other hand, is a ubiquitous feature nowadays. That's great to see.

|                      | TextNut  |  Typora   | HarooPad |  Quiver  |  Atom   | Sublime  |  Texts   |
| :------------------- | :------: | :-------: | :------: | :------: | :-----: | :------: | :------: |
| WYSIWYG              | **Yes**  |  **Yes**  |    No    |    No    |   No    |    No    | **Yes**  |
| Easy to Use          | **Yes**  |  **Yes**  | **Yes**  | **Yes**  |   No    |    No    | **Yes**  |
| Both Mac & Linux     |    No    |  **Yes**  | **Yes**  |    No    | **Yes** | **Yes**  |    No    |
| Free / OSS           | No ($25) | No (Beta) | **Yes**  | No ($10) | **Yes** | No ($70) | No ($20) |
| Auto-save            | **Yes**  |  **Yes**  | **Yes**  | **Yes**  | **Yes** | **Yes**  | **Yes**  |
| Work Directly in .md |   No*    |  **Yes**  | **Yes**  |   No*    | **Yes** | **Yes**  | **Yes**  |
| Leaves TOML Intact   | **Yes**  |  **Yes**  | **Yes**  | Unknown  | **Yes** | **Yes**  |    No    |

\* = TextNut can open and edit a .md file, but the WYSIWYG aspect only works when it is "imported" to the proprietary format and edited there, then exported back to Markdown. Quiver has similar focus on Notes and a similar weakness in working in .md directly: basically it's a less capable TextNut.

I like [Texts](http://www.texts.io), but it breaks the "front matter" on a Hugo post, so that's a deal-breaker. [HarooPad](http://pad.haroopress.com), besides having a lack of  documentation in English and development that has been dead for a couple of years, is pretty robust. If it could only offer WYSIWYG editing I think it would have been my choice.

So the overall winner for me is [Typora](https://typora.io). Eventually it will leave beta, and they'll charge for it, but hopefully it's something reasonable. Until then, it's free!

## QuickLook for .md Files (MacOS)

I am all about using QuickLook in MacOS. You just hit space bar on a file in Finder and you get a perfectly good read-only peek at the file. But it doesn't handle Markdown (as plain text, let alone as a rendered view). Fortunately [someone made a QuickLook generator](https://github.com/toland/qlmarkdown/releases) you can install using Homebrew: `brew install Caskroom/cask/qlmarkdown`.

Now you're ready to work with Markdown files!

## It's Not All Roses Though: Behind the Scenes on This Post

The rendering of your content doesn't *always* turn out the way you envisioned. When this happens, it's either your Hugo theme, or the Markdown renderer that is to blame. Unfortunately, this might mean rolling up your sleeves and fixing CSS, as I had to do in order to get the table above to look decent. Hopefully this is a one-time thing. I had to go into the purehugo theme subdirectory, and edit `static/all.min.css` within that.

Secondly, when editing Markdown to embed an image from a local directory (as I've done above), Hugo requires you to put the files in `/static` and then in the Markdown you specify a relative path *without* a leading slash, such as `![MarkdownEditors](img/markdownEditors.png)` (actual path of the image in the source tree is `/blog/static/img/markdownEditors.png`) and Hugo will copy it to the publishdir during rendering. Because of this, you can't actually see the image in your Markdown editor, which sucks, and your source tree will have two copies of the file, which also sucks.
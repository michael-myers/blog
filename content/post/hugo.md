+++
Tags = ["web","blogging","Hugo", "Jekyll", "YAML", "TOML"]
Description = "Initial impressions on the static site generator, Hugo"
date = "2017-02-03T19:07:12-05:00"
title = "Hugo, the static site generator"
Categories = ["web","blogging","Hugo"]
+++

## Hugo, a Static Site Generator

In my last post, I covered the rationale behind using a static site generator. Static site generators are not just for creating blogs. They can also be used to create online resumes, company sites, online documentation, etc.

The default choice for static site generator is Jekyll, which has the most support, but it's troublesome to install and use. Hugo is a popular alternative that is easier to install, and faster to work with. It's implemented in Golang, a.k.a. Go. This means it is written in a statically compiled language (The Best Kind) and is *completely dependency free.* Dependency hell is the bane of my existence. It's like work that you have to do before you can start working. Anyway, let's look at how to get started.

## Hugo Install Process (MacOS)

This is so simple, and its simplicity is the reason why I went with Hugo after trying the more popular Jekyll, which was a mess.

```sh
brew update && brew install hugo
hugo new site myBlog
cd myBlog
git clone https://github.com/dplesca/purehugo.git themes/purehugo
echo "theme = purehugo" >> config.toml 
```

Creating or customizing themes is beyond the scope of this post, but what we are doing here is "installing" a pre-baked Hugo theme, and then setting it as our default.

## Hugo Workflow: Drafting & Publishing a Post (MacOS)

In order to create a new post for your blog:

```sh
cd myBlog
hugo new post/myReviewOfHugo.md
open content/post/myReviewOfHugo.md # write the post in your text editor

# Optional: launch a local webserver, give it a sec, and preview the blog
hugo server & sleep 2 && open http://localhost:1313/blog/
killall hugo # because we left hugo running in the background there
```

While the server is running, you can actually continue to edit the post in your editor. The server will live update the view in your browser. This is optional, but it will verify that everything will look correct when you publish.

When you're satisfied, you can generate the actual web content to disk, and publish it. The following steps assume you are using Github Pages, so the publish is made using a `git` push.

```sh
# You must already have a GitHub project, and in its settings page, and have set the GitHub pages to "master branch / docs". In this example, the project name is "blog".

# These are the one-time Hugo steps:
echo "publishDir = docs" >> config.toml
echo "baseURL = https://myname.github.com/blog" >> config.toml

# These are the one-time Git steps:
rm -rf themes/.git # delete existing git files so they don't interfere
git init  # turn this directory into a git repo
git remote add origin https://github.com/myname/blog.git

# These are the only steps needed every time you publish new content:
hugo  # this generates HTML + JS + CSS under the publishdir (blog/docs/)
git add -A
git commit -m "Add a blog post about whatever."
git push
```

That's all there is to it, although you can always use a different Git client if you don't like the command line. I sure as hell don't like it (I use Atlassian Sourcetree) but it's up to you.


Post Metadata: WTF is "Front Matter" ?
--------------------

In each post (each Markdown file), there is some metadata in a header at the top of the file, called "front matter." Jekyll was the first to introduce this concept (in name, at least), but it is common across other static generators now. Hugo lets you write front matter in YAML, JSON or TOML (the default). If you've worked in web development surely you've heard of JSON, but now you may be asking WTF is YAML and TOML?

These are syntaxes invented specifically for controlling the settings of static site generators. It seems to be a case of "reinventing the wheel" of [INI files](https://en.wikipedia.org/wiki/INI_file), which have been around for decades. Basically, a config file. Key-value pairs. Associative array. Hash table (please don't shorten it to just "hash," words have meanings, know the difference). Dictionary. They're all basically the same thing. YAML started in 2009 or so, as a minimalist-syntax alternative to JSON, which itself was a minimalist alternative to XML. [We'll get this right some day](https://en.wikipedia.org/wiki/Comparison_of_data_serialization_formats).

The CEO of GitHub and inventor of Jekyll, probably high on the smell of his own farts, in 2013 decided that YAML needed to be even more minimal, and renamed this idea after himself ("TOM"), and thus was born TOML, which primarily because of the fame of the creator has now spread [to a few other projects](https://github.com/toml-lang/toml#projects-using-toml). Thus, we have minimalized almost all the way back to INI files (except now it has been "standardized"). Progress.

Oh and by the way, none of these *are actually markup languages at all*. They just aren't. The insistence on propagating the use of the acronym letters -ML for config file formats is basically an inside joke at this point.

The takeaway for me is that in the mid-2000s it became fashionable to ditch braces and brackets in all syntax for everything, in favor of careful indentation. Thus returning to the fashion of the 1970s and FORTRAN. You know what's popular *today*, though? Look at Go, Rust, and Swift. Yea that's right, compiled languages with curly braces are back again. [Urge to kill risinnnnnnng.](https://www.youtube.com/watch?v=sX0CbA4718A) All right, deep breaths.

Anyway, within this "front matter," you can define tags and categories, timestamps, and titles for every post. For examplte, the front matter for this post was defined as such:

```toml
+++
Tags = ["web","blogging","Hugo", "Jekyll", "YAML", "TOML"]
Description = "Initial impressions on the static site generator, Hugo"
date = "2017-02-03T19:07:12-05:00"
title = "Hugo, the static site generator"
Categories = ["web","blogging","Hugo"]
+++
```

You can also set optional variables like a publish date in the future (Hugo will not render it to the content directory until this date), or an alias (if you want to forward visitors from another URL to this post instead).

The configuration file for your Hugo site, `config.toml`, is also in this syntax.

That more or less covers the basics of Hugo, and static site generators like it. My next post will be about Markdown (an *actual* markup language).
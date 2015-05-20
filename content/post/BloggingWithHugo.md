+++
comments = true
date = "2015-05-20T18:09:40Z"
draft = false
image = "images/dollySods_cover_CP.jpg"
slug = "BloggingWithHugo"
tags = ["blog", "design"]
title = "Blogging With Hugo"

+++

### Hello World

I have finally decided to do something with this website of mine.
For a while it's been sitting here with a static page on top, but with some spare time
and a bit of desire for a challenge, I have updated the site. The goal will be to
continue to blog on it, while adding additional content to the site.

My main focus will be some of my random adventures in life, but at the same time I'll
be blogging from time to time about my trials of maintaining this site.

I built the site using [Hugo](http://gohugo.io) and picked up a theme called
[Casper](https://github.com/vjeantet/hugo-theme-casper). It's a little tricky
since I'm going about the blog in an odd fashion. First I edit files on my computer
at work/home/phone. Then I use git to push the content to github. Then, when I get
a chance, I'll use git to pull the changed content on my VPS with Digital Ocean. Once there,
I use Hugo to generate the static pages for the content, which is then copied
to a separate location which is where NGINX picks up the content and puts it
on the web.

```Steps
hugo new -t casper post/BloggingWithHugo.md
** Write blog post in notepad **

git push https://github.com/elShambler/beingMas_blog

** Log in to VPS **
git pull
hugo -t casper
cp -r public/* </location/for/nginx/host>
```

Convoluted and most likely unnecessary. Because this is my first real experience with
maintaing a webserver, it is a constant learning process. With any luck, I will have
greater efficiency in the future. For now, this method will have to work.

Next thing to try is symlinks and/or rsync to just copy the files directly for nginx
to serve.

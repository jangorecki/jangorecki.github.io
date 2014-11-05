---
layout: post
title: Hello World
tags: jekyll knitr github.io
---

Hi,  
I've just setup my new blog. Most probably it will be R related, data processing, etc.  

Some comment about the technology in this very first post.  
Posts on the blog are `md` files. These can be written by hand or produced from `Rmd` file which includes executable R code chunks.  
When a `YYYY-MM-DD-title.md` file is pushed to your github repository it will automatically regenerate your github pages (`github.io`) using [jekyll](https://github.com/jekyll/jekyll) (static sites generator).  
The easiest way to setup jekyll can be just `fork` the [Jekyll Now](https://github.com/barryclark/jekyll-now), edit title, etc. in `_config.yml` and start pushing (or copy+paste content) `md` files into `_posts` directory.  
It can serve rss, disqus, google analytics out of the box and it can be deeply customized.

```r
knitr::knit("2014-11-05-Hello-World.Rmd")
```

Jan

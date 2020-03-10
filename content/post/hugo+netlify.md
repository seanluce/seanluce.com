---
title: "Hello Hugo and Netlify"
date: 2020-03-09T15:57:54-04:00
categories:
- meta
tags:
- hugo
- netlify
- blog
keywords:
- blog
- hugo
- netlify
#thumbnailImage: //example.com/image.jpg
---
Hello and welcome! I guess this is seanluce.com 2.0. 3? 4? I've lost count. I felt like it was time for a frest start. So this is it. A clean slate. With a new start comes a new content platform. I kicked the tires on a few static site generators like [Jekyll](https://jekyllrb.com), [Gatsby](https://www.gatsbyjs.org), and [Pelican](https://blog.getpelican.com) (you can see a pretty exhaustive list at [staticgen.com](https://www.staticgen.com) if you are curious). I eventually landed on [Hugo](https://gohugo.io) for static site generation and [Netlify](https://www.netlify.com) to build, test, and deploy. I am still getting used to the workflow, but it goes something like this:

{{< highlight plain-text >}}
1. $ hugo new post/title+of+new+post.md
2. $ macdown ./post/title+of+new+post.md
3. ## Write that blog post. (This is usually the difficult bit.)
4. $ git add *
5. $ git commit -m "adding a new post"
6. $ git push</code>
{{< / highlight >}}

After that final step, Netlify automatically detects the changes to my github repo and kicks off the build. Within a few seconds, the new content is live.

I can also test locally by invoking Hugo's built in web server: {{< highlight plain-text >}}$ hugo server {{< / highlight >}}

This allows me to easily make sure everything looks good before pushing to build. The entire process is really clean. All of the sites content is stored on github in regular text. It just feels light and airy...like clouds. Cute, fluffy, clouds :cloud:.
<!--more-->




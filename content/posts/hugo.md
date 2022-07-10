---
title: "Hugo"
date: 2022-07-08T20:16:37+02:00
draft: false
toc: false
images:
tags:
  - untagged
---

## Hugo in 5 minutes

**[Hugo]**(https://gohugo.io/) is a static site generator written in Go.

What about **Wordpress**? Done that.. What about building a blog using **Django**? Done that.. 

I picked **Hugo** because I don't want to mess around with hosting and just focus on writing content.
For me static site generators (like **Hugo**) works out of the box, but still gives me the possibility to modify things if needed. 

GitHub provides free and fast static hosting over SSL and at the same time automates the whole process. Yeeah ! :) 

## Deploy a Hugo blog in less than 10 steps 

1. Get a [GitHub]("https://www.github.com") Account
2. Create a GitHub repository and name it `<username>.github.io.`  
3. Create a Hugo project `hugo new site . --force` 
4. Pick a (Hugo) theme from [here](https://themes.gohugo.io/)
5. Add content to your new blog. `hugo new posts/hugo.md`
6. Generate the blog by running `hugo`
7. Push to GitHub
8. Enable **GitHub Pages**
9. Visit your published blog at https://<username>.github.io


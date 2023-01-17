---
title: "A blog is born"
date: 2023-01-15T17:39:13-05:00
tags: [blog, markdown, pi, website]
draft: false
---

Hello, world! Probably the most used sentence to start a blog.

I bought this domain in 2019 promising myself I would launch a blog
someday. I have been trying on and off to get something running. I really wanted
to build a CMS with React or Vue and Python/Django from scratch just to get to
know the inner workings, open source it and then host it at this domain as my blog.
[This project](https://github.com/rob32/dev-case) is a perfect example of what I wanted to achieve.
This would run on my Raspberry Pi 4B at home. I was all excited. But then, life happened.

Fast-forward 4 years, and I had nothing except the Pi running with a [basic landing page](https://github.com/viseshrp/portfolio). I worked on several open source tools in
that time which were all really necessary for me to make my life easier. It turned out, a CMS was not one
 of them. There are tons of tools out there that do the job very well already. Moreover, I realized
 I did not even need a CMS. All I needed was a static site generator and a bunch of `md` files to
 get a website like this one.

 I looked at a bunch of websites that did this already.
 [Hugo](https://gohugo.io/) and [Astro](https://astro.build/) were good options. I settled on Hugo purely for the number of GitHub stars and the [Ink](https://github.com/viseshrp/hugo-ink) theme. How superficial of me.

 Hugo turned out to be great. I had a website running within minutes. Once I created a simple stub, I made a [Docker container](https://github.com/viseshrp/website) out of Hugo and deployed it on my Pi.
 Port-forwarding and everything were already setup. I modified my proxy to point to the container and finally, my blog was born.

 I expect this blog will mostly be for brain dumps of things I work on, to help strangers on the internet who run into similar problems and also for my future self but it could also contain ramblings about my life in general.

Great time of the year to start something new, [innit](https://github.com/viseshrp/website/commit/b58b191c4977ef847891279f3aaecb9c4a1ba43b)?

---
date: 2025-03-17T02:00:00-05:00
draft: false
params:
  author: Greg Sarjeant
title: Webmentions with webmention.io
url: 2025/03/17/eli5-webmentions
tags: [indieweb]
weight: 10
---
I can't wrap my head around webmentions. I thought I understood them. Then I redid my site's URL scheme and they all broke, which made sense, but when I thought about what I needed to do to fix them I couldn't figure out how they had ever worked in the first place. I get it again, but I know I'll forget the next time something happens, so this is a very basic primer that will hopefully help it stick. In the worst case, I can refer to next time.

I use [webmention.io](https://webmention.io) to receive webmentions. I've started to think of the process in terms of returning books to the library. I may come up with something better, but this is at helping me keep it in my head.

When you return a book, you go to the library and look for the book drop off. You find a sign that points you to the book drop and you drop your book in. Sometime later, a librarian takes the book from the book drop and puts it on the shelf it belongs on. Then when someone goes to the shelf, they find the book.

When someone wnats to leave a webmention on my site, they go to my site and look for my webmention endpoint. They find a `<link rel="webmention">` tag that points to webmention.io and drop the webmention there. Sometime later, a bit of javascript takes the webmention entry from webmention.io and puts the webmention on the page it references. Then when someone goes to the page, they find the webmention.

It's not perfect. I'm taking some liberties with timing and what it means to put something on a page. But it works for me.
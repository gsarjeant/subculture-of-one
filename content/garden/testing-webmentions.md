---
date: 2025-02-15T22:00:00-05:00
draft: false
params:
  author: Greg Sarjeant
title: Testing Webmentions
url:  2025/02/15/testing-webmentions
aliases:
  - /forest/testing-webmentions
weight: 10
tags: [indieweb, tests]
related:
  - author: aaronpk
    url: https://aaronparecki.com/2018/06/30/11/your-first-webmention
---

This is just a quick test of webmentions, based on Aaron Parecki's [your first webmention](https://aaronparecki.com/2018/06/30/11/your-first-webmention) post. The webmention itself is displayed beneath the post content, after the tags. Once this works, I'll try automating the sending.

Okay, now I've tried modifying the post template so that all the necessary markup is where it should be in the post. I also don't treat webmentions in long-form posts as replies. I wonder what will happen now.

Ok! It worked as expected. This post fell down to the "Other mentions" section of Aaron's page. But that's okay - it's what I wanted. I think more often than not I'm going to want to mention other pages because they're related to whatever I'm writing about. I'm not generally going to be posting a direct reply to someone else's post.

I also have realized that there's actually nothing special about this kind of generic webmention. Any link to another site can be a webmention, as long as that site supports them.

You can get fancier and do things like replies with additional markup (e.g. `in-reply-to`), but that's not required.

But now that I've spent a day messing around, I like the idea of a "related posts" footer section, even if it's sometimes redundant, so I'm sticking with it.

### 2025-03-15

It just struck me that the helpful thing to do here would be to actually write up what I'm trying to do and how I did it so if anyone else ever wants to do the same, it might help. But of course I had this thought just as I'm about to leave town for a couple weeks. With luck, this note will remind me to do that on some quiet evening overseas.
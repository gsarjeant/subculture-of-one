---
date: 2025-01-17T00:00:00-05:00
draft: false
params:
  author: Greg Sarjeant
title: On the importance of friction
aliases:
  - "/2025/01/17/on-the-importance-of-friction.html"
weight: 10
tags: [trees, indieweb-carnival]
---


Nothing happened. And then everything happened.

In 2004, I bought a new laptop. It was a Gateway 200X (Do you remember the cow boxes? I remember the cow boxes). I was a fairly serious Linux user at the time, so I installed Gentoo Linux on it. Just about everything worked right away, which was both surprising and a little disappointing. It hadn't been too long since things like USB support meant you were recompiling your kernel, so rebooting from the installer into a fully functional environment left me feeling a little empty. It was a different time.

I think this was my first laptop. So for the first couple of weeks I shut it down from the power menu the way you (or at least I) shut down a desktop. It never crossed my mind to hit the power button to shut it down. One evening, I decided to try that. I hit the power button ... and nothing happened.

My heart leapt. Here was a thing that didn't work! It also was right around Christmas, so I was about to have a bunch of time off. Ideal conditions to figure out how to shut down my computer from the power button. What better way to spend a holiday?

What followed was an odyssey that in many ways shaped the rest of my life. I'll skip a lot of the details about how I got it working. Most of them are irrelevant today, and the specific steps are beside the point. What's important is the fact that I had to go searching.

I learned that there was a specification for power management (ACPI), and that it was published and I could read it. I learned that most laptops at the time didn't follow it terribly strictly. I learned that there was a programming language for it, and that I could fix my own broken power management by using it. I learned that I had to patch the linux kernel to fix some things and I even learned how to write a kernel patch that would make anybody who's written a real kernel patch write me a strongly worded letter.

After about a week of research, trial and error, qualified successes, and setbacks, the moment came when I hit the power button and watched my laptop gracefully shut down. I spent the next 15 minutes just turning it on and turning it back off again. I showed my wife, who graciously feigned excitement. It was immensely gratifying.

I'd seen enough questions about ACPI on the Gentoo Linux forums that I wrote a [guide](https://subcultureofone.org/2025/01/17/an-act-of-preservation.html). This was also a learning experience! When I get into writing something, I have a lot of fun. I like to tell little anecdotes, make bad jokes, go off on the occasional tangent. It turns out that when someone is reading something to fix something that's broken, they don't want any of that! I edited my guide constantly, revising it based on feedback. I learned a lot about what makes good technical writing and what doesn't.

I also got to help a lot of people directly. Every time I was able to help someone fix their computer's ACPI implementation, I felt genuine joy. It was even more gratifying than getting it to work for myself.

Friction hampers productivity, but it stokes curiosity. Productivity gets things done, but curiosity lets us ask whether the thing we're doing should get done. I once had a physics teacher who said that without friction, we'd slide off the world. As I see how effortlessly people bring harm to one another with the frictionless technologies we've built, I feel that we're doing exactly that. I long for the friction that can stop the slide and let us remember what it is to be rooted.

Had that power button just worked all those years ago, I'd have missed out on all of this learning. But because it didn't, I gained a deeper understanding not just of how my laptop worked, but of how _I_ work. Each question I asked led me to another question. As I started to understand my specific problem, I started to ask different questions. What other things are broken in my ACPI implementation? Can I fix those? Are other people having similar problems? Can I help them?

The answers to those questions have shaped much of the last 20 years of my life. They've changed my career. They've introduced me to new hobbies. They've brought me to communities both physical and virtual. I would not be the person I am today if I hadn't asked them.

And it all happened because nothing happened.

_This is my submission for the January, 2025 [IndieWeb Carnival](https://vhbelvadi.com/indieweb-carnival-friction), hosted by V.H. Belvadi._


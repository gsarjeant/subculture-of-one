---
date: 2025-05-12T00:00:00-05:00
draft: false
params:
  author: Greg Sarjeant
title: SSH Key-based authentication
url: 2025/05/12/eli5-ssh-key-based-authentication
tags: [eli5ium]
weight: 10
---
I never used to be able to get SSH keys to work. I was always mixing up where the public key goes and where the private key goes. For some reason the model never clicked. Now it makes sense, but I've been thinking about how I might have explained this to confused Greg back in earlier times.

I think physical keys and locks work well - better than the physical analogs of digital terms often do.

Your private key is the key. Just like a physical key, you want to keep it on your person. With a physical key, this means it stays in your pocket or your handbag. For SSH, your private key stays on your local machine (your laptop or whatever). It lives on the system that you are connecting *from* and it never leaves. If someone gets a hold of your physical key, they can open any lock that it fits. Likewise, if someone gets a hold of your private key, they can connect to any SSH server that accepts it.

Your public key is the lock. It goes on the server you're connecting *to*. It governs access to a remote server in much the same way that a physical lock governs access to a building. When you approach a building, you're confronted with a challenge: does your key open this lock? You pull the key out of your pocket and if it opens the lock, then you can get into the building. Similarly, when you connect to an SSH server, you're confronted with a challenge: does your private key work with this public key? If it does, then you're granted access to the server. If not, then you aren't.

This also works to explain why it's okay to distribute your public key. It's the same as putting your lock on a bunch of doors. The lock doesn't make the door any less secure. It just makes it possible for someone to enter if they have your key. As long as nobody else has your key, nobody else can open that door. And that's why you *don't* distribute the private key. 

Once someone has your key (private key), they can get into any building (SSH server) with a lock (public key) that it fits.

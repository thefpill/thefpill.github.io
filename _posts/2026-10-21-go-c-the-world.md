---
layout: post
title: "Start of my journey with Go lang in C-world"
date: 2026-10-21 11:30
tags: [Golang, Windows Internals ,Detection Engineering , DFIR,  MemoryForensics, Win32API, ThreatHunting]
---

After the brief introduction to this blog and project of mine, there's one thing I forgot to mention - I am not a developer. I do not have any formal education or experience, so please bear with me.
With that said it's not a surprise I hit a wall sooner then I expected. 

Go is a language that has many strengths .. I'm sure - but one might say I chose the wrong tool for the job. It's not to say that there aren't any packages for the **Win32 API** - there's `x/sys/windows`. 

But let's just say that many of the lower-level operations I would want to do aren't native. But that is not the point and we will tackle that in later posts (maybe)

My beginner mistake was thinking I could create arbitrary structs in Go, just pass a pointer to them, and everything would magically line up.

Oh boy Was I wrong. 

Naively, I thought that embedding `PROCESSENTRY32` - a.k.a. copying the fields with the appropriate types, but also adding my own custom field - would work.

### the reality check 

When you pass a pointer to a struct into a syscall, the Windows API checks the `Size` field you specify. So, basically, if you pass a struct - as I did - with padded fields or alter its alignment, the system call may reject it.

Also, if you reorder the fields in your struct, the system call might still accept it - but the data wouldn't make sense, since Windows will just write data at predefined offsets.

### so what did we learn ? 

Don't mess with memory layouts for your convenience. Collect the data first, then map it to your custom structures.
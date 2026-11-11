---
layout: post
title: "Breach Bench"
date: 2026-11-11 08:30
tags: [Python, JS,Adversary Simulation, Architecture Playground ]
---

Hey guys,

to take a break from our regular program -  I am happy to announce a little project I thought some of you could find useful.  ->  [Breach-Bench](https://breach-bench.vercel.app/)

Credit where credit is due - while back I came across a project (`https://github.com/JoaoGeraldes/system-design-playground`)
That lets you - *Build a system design on a board, send up to 10 million daily users through it, and see which part breaks first and how to fix it.*

So that got me thinking - could this be applied in cybersec ?  

*To be completely honest I didn't build it myself all credit goes to our AI overlords - my philosophy is that if I want to learn something I build it myself , but this seemed as more of a presentation / design task then system deep dive.*

So after some trial and error I got the **breach-bench** working - the simplest explanation is -
Its a playground that lets you design somewhat accurate representation of your environment / assets  and then you can  run campaigns that were observed being used by different actors ( everything is mapped to  `MITRE ATT&CK` ) 

The `breach-bench` gives you the ability to not only design but also see the different views - what a defender might see, what actually happened , coverage matrix  to better understand where the gaps are. 

You are also able to add / modify existing scenarios. Experiment with different EDR  settings for logging or just play around. Once you are done you can export your designed environment so you can import it later. We dont store anything about any session - ever. 

But as always a picture ( or in this case a experience ) is more then thousand words - so give it a try for yourself . 



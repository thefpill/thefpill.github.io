---
layout: post
title: "The traceforge"
date: 2026-10-13 11:30
tags: [Golang, Windows Internals ,Detection Engineering , DFIR,  MemoryForensics, Win32API, ThreatHunting]
---
## Intro
I always believed that you learn the most by doing. Thats why I started my newest project, TraceForge.
The goal is simple - learn about Windows internals in the practical sense. Im not claiming that I have the know-how or the expertise for a project of this scope - but hey.. you never know.

When coming up with the idea itself, I of course had long conversations with my advisor on the topic - the almighty AIs.
I know, I know .. everybody and their mom is using AIs without second-guessing anything. But here is the disclaimer - everything in this project is written by hand - even this post. But! I did use AI to bounce ideas and concepts around.

Moving on - after the initial brief, I have settled on this dependency/workflow graph that would serve as a rough guideline.
Each "bucket" would be its own topic, a research subject that would be interlinked with coding challenges.

                         ┌──────────────────┐
                         │ 0. Architecture  │
                         └────────┬─────────┘
                                  │
                 ┌────────────────┼────────────────┐
                 ▼                ▼                ▼
        ┌────────────────┐ ┌──────────────┐ ┌───────────────┐
        │ 1. Process     │ │ 2. Filesystem│ │ 3. Networking │
        │ internals      │ │ & artifacts  │ │ & sockets     │
        └───────┬────────┘ └──────┬───────┘ └───────┬───────┘
                │                 │                 │
                └─────────────────┼─────────────────┘
                                  ▼
                         ┌──────────────────┐
                         │ 4. Persistence   │
                         └────────┬─────────┘
                                  │
                    ┌─────────────┼──────────────┐
                    ▼             ▼              ▼
             ┌───────────┐ ┌──────────┐ ┌──────────────┐
             │ 5. PE/ELF │ │ 6. Memory│ │ 7. Telemetry │
             └─────┬─────┘ └─────┬────┘ └──────┬───────┘
                   │             │             │
                   └─────────────┼─────────────┘
                                 ▼
                       ┌────────────────────┐
                       │ 8. Detection Engine│
                       └──────────┬─────────┘
                                  │
                       ┌──────────┴──────────┐
                       ▼                     ▼
                ┌────────────┐        ┌─────────────┐
                │ 9. Timeline│        │ 10. Hunting │
                └────────────┘        └─────────────┘
				
## Objective 

The goal of this project is as I already mentioned - is to build a tool that would help better understand how different components interact on deeper level.
To see the bigger more holistic picture when investigating and maybe to help some of you overcome obvious and maybe not so obvious hurdles if you would attempt something similar. 

I hope to cover some indepth topics but also some small blunders/wins I make along the way.

With that - lets get to it. 
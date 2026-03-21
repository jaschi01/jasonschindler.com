---
layout: post
title: "Twenty years in: how I got here"
date: 2026-03-21
tags: [Career, AutoCAD, AutoLisp, Design Master]
excerpt: "A QBasic side-scroller, a kitchen wall covered in pseudocode, and twenty years of building software for engineers. This is how I got into development — and what's kept me here."
---

This June marks twenty years since I wrote my first line of professional code. I've been meaning to start a blog for a while, and it felt right to kick things off with a bit of context — who I am, how I got here, and what kind of problems I find interesting. Future posts will get more technical. This one is just the backstory.

<!--more-->

## A QBasic class and a side-scrolling shooter

I grew up in an area where computer science education wasn't exactly abundant. In the late 90s, most schools weren't offering much in the way of programming, so I was lucky that mine offered anything at all. Junior year I landed in a class built around QBasic — not exactly the most glamorous language, but it was enough.

By the end of that class I had built a 2D side-scrolling shooter. It wasn't much to look at, but it *worked*, and something about that — the idea that you could describe a set of rules and watch a machine carry them out — got its hooks into me immediately. I was done for.

Senior year I followed it up with Visual Basic and C++, and by the time I graduated I knew what I wanted to study. I went on to computer engineering in college, and the rest, as they say, is history.

## Finding a niche in AEC software

I've spent my career at Design Master Software, building add-ins for AutoCAD and Revit that help mechanical, electrical, plumbing, and fire protection engineers do their work. It's a niche corner of the software world — most developers have never heard of the AEC space, let alone the tooling that lives inside CAD platforms — but I've come to love it. The domain is genuinely complex, the users are technical and demanding, and the feedback loop between software and real-world building systems keeps things interesting.

I started on the AutoCAD side of the house, working in AutoLisp and the AutoCAD API, before eventually moving into C# and the Revit API as BIM became the direction the industry was heading. Both platforms have their own personalities, their own quirks, and their own ways of humbling you when you think you've figured them out.

## The kitchen wall

Early in my career I was handed a problem that I still think about: build a way to merge two Design Master projects together.

On the surface that sounds straightforward. In practice it was anything but. A Design Master project lives in two places simultaneously — an external database and the AutoCAD DWG file itself, which acts as its own kind of database. Merging two projects meant reconciling both, consistently, without losing data or corrupting either side. The relationships were deeply nested, the edge cases multiplied fast, and the whole thing had to be implemented in AutoLisp.

I had recently painted a wall in my kitchen with chalkboard paint — one of those home improvement ideas that seemed more practical than it turned out to be. It ended up being perfect for this. I mapped the entire recursive structure on that wall in pseudocode: how each entity related to others, where the merge logic needed to branch, how conflicts would be resolved. Floor to ceiling, edge to edge.

Looking back, I'm proud of what came out of it. Not because it was flashy — AutoLisp is about as far from flashy as software gets — but because the design was clean. The recursion was elegant. It solved a genuinely hard problem in a way that held up. That's still what I'm chasing.

## What this blog is for

Twenty years in, I still find this stuff interesting — which I take as a good sign. These days most of my work is in C# and the Revit API, with occasional detours into whatever corner of the stack a problem drags me into.

This blog is where I'll write about the things I run into: Revit API patterns that took me a while to understand, AutoCAD development lessons learned the hard way, C# approaches that worked out, and the occasional war story. If you're building in this space too, I hope some of it is useful.

More soon.

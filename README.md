# jasonschindler.com

Personal portfolio and dev blog built with Jekyll, hosted on GitHub Pages.

## Local development

```bash
gem install bundler
bundle install
bundle exec jekyll serve
```

Then open http://localhost:4000

## Adding a blog post

Create a new file in `_posts/` named: `YYYY-MM-DD-your-post-title.md`

Front matter template:
```yaml
---
layout: post
title: "Your Post Title"
date: 2026-01-15
tags: [Revit, C#]
excerpt: "One or two sentences shown in the post list."
---

Your post content in Markdown here.

<!--more-->

The rest of the post...
```

## Deployment

Push to the `main` branch — GitHub Pages builds and deploys automatically.

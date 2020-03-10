---
title: "Adventures in Starting a Blog"
date: 2020-02-29T16:42:34-05:00
draft: false
description: "My personal journey setting up a blog."
categories:
  - "Side Projects"
tags:
  - "Blog"
  - "Hugo"
  - "Gatsby"
  - "Github Actions"
  - "Netlify CMS"
  - "Forestry.io"
lead: "A million little decisions."
---

Like a moth to the flame I keep coming back to the idea of maintaining a personal blog. I’ve spun up a [Jekyll](https://jekyllrb.com/) blog more than once and wrote some content only to abandon it days later. Until a bit ago cbelsole.github.com was still up and running. Since I haven’t touched it for 4 years it stood as a testament to set it and forget it infrastructure.

So like, why do this again? Since my Jekyll days static site generators have gotten a lot more interesting. I use Gatsby (JS) professionally which has the ability to do plug-and-play server-side rendering. This is a huge advantage since you can cut perceived load time to a minimum by pre-rendering all of your react pages and ship the static site async after the page loads. Hugo (Go) promises to render pages in 1ms making generating thousands of pages fast. Being web developers, we owe it to ourselves to explore the tools of our field so that we can craft nuanced solutions for professional and personal projects.

# Goals

For my static site experiment I wanted to set a few goals with a focus on creating content and not configuration:

1.  I want a static site generator that works out of the box without a lot of configuration

2.  My strengths don't lie in frontend development. So I need a template to work off of.

3.  Blog posts should be easy to create. Experiment with a CRM for creating posts and managing content.

4.  I want the hosting platform to be bulletproof so that I can leave the blog up for years without worrying about it being down.

5.  I want to accomplish these goals on the cheap.

# Picking a framework

If we check [staticgen.com](https://www.staticgen.com/) there are currently 281 options for creating a static site. That’s too many. So I shortened the list to [Gatsby](https://www.gatsbyjs.org/) (JS) which I have production experience with and [Hugo](https://gohugo.io/) (Go) because I mainly work in Go these days and you get a lot of tooling for free.

## Gatsby

Gatsby's ability for server-side rendering is appealing especially if you have a few pages that are going to be visited frequently. For this project though I want something that works out of the box and requires very little upkeep. If you’ve ever worked with JS in production you know that dependencies can cause a maintenance nightmare if you 1. Don’t lock them down and 2. Don’t keep them up to date. For example, try upgrading your version of React after being 2 major versions behind.

## Hugo

I have been following Hugo for a long time. I experimented with it once when it was first released and it lived up to its promise of rendering pages quickly. At the time the themes available were sparse and my strengths don’t lie in front end development. Since then their themes folder has grown to 299 themes as of the writing of this post. They all range in their capabilities like social integration and Google Analytics, but some of them look straight up professional. I’m looking at you [hugo-academic](https://github.com/gcushen/hugo-academic).

> **Protip:** Github user TrentSPalmer created a script to rank Hugo themes by stars [hugo_themes_report](https://github.com/TrentSPalmer/hugo_themes_report).

What about dependencies? Installing Hugo comes with a long list of dependencies, but since the only thing I installed is Hugo I can assume that they will manage that list for me. So upgrading Hugo should be fairly straightforward. That coupled with Go's promise to maintain backward compatibility lowers the tooling and maintenance costs. I did read a few cases where Hugo did not perform the same version over version, but I’ve lived a lifetime maintaining JS applications. So if I have to deal with the occasional upgrade bug from one dependency rather myriads I am fine with it.

# Setup

```bash
# create and enter directory
mkdir cbelsole.com && cd cbelsole.com

# create a new go module
go mod init github.com/cbelsole/cbelsole.com

# install hugo and lock version
go get github.com/gohugoio/hugo

# create new hugo site
hugo new site cbelsole.com && mv cbelsole.com/* . && rm -rf cbelsole.com

# add the mainroad theme
git submodule add https://github.com/vimux/mainroad themes/mainroad
```

# Content Management System Rabbithole

To manage my blog tags and content I looked into [Forestry.io](https://forestry.io) and [Netlify CMS](https://www.netlifycms.org) since both of these had plug-and-play content tools that integrated with Github. The idea here is to make managing content easier by using a UI rather than editing post metadata manually.

> **TL;DR** it did not work out. The need to integrate with their specific sites outweighed the benefits of a CMS, especially with so little content.

## Netlify CMS

The experiment started out with a lot of promise since in their words:

> The folks at Netlify created Netlify CMS to fill a gap in the static site generation pipeline. There were some great proprietary headless CMS options, but no real contenders that were open source and extensible—that could turn into a community-built ecosystem like WordPress or Drupal. For that reason, Netlify CMS is made to be community-driven, and has never been locked to the Netlify platform (despite the name).

> If you hook up Netlify CMS to your website, you're basically adding a tool for content editors to make commits to your site repository without touching code or learning Git.

The downside here is that plugging in the CMS requires you to use their identity service for OAuth and that means you have to deploy via Netlify. This is not something I wanted to do since the price per usage is good compared to Zeit but not as good as AWS. You can host your own identity server, but that would necessitate standing up a server which would also increase costs.

## Forestry.io

Forestry.io is another Github based CMS provider that allows you to plug it into your website pretty seamlessly. When I got it setup I took a look at what was in front of me. I have a tag manager and a markdown editor. As the sole writer with only a single post this seemed like a premature optimization. Maybe when I am writing with multiple people with a more complex use case adding a CMS will be more of a need than a nice to have. For now, with [stackedit.io](https://stackedit.io/) I have a good markdown editor and eventually I'll come up with a set of tags.

# AWS Time

[This guide](https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-serve-static-website/) describes how to create a static s3 site with HTTPS via CloudFront. After the $12 of registering the domain name with my current usage AWS costs me around $1.03 a month. That's well within what I am willing to pay for hosting.

1 github workflow triggered on pushes to master later:

```yaml
name: build deploy hugo

on:
  push:
    branches:
      - master

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo with submodules
        uses: actions/checkout@v1
        with:
          submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "0.65.2"

      - name: Build Static Site
        run: hugo --minify

      - name: Setup AWS CLI
        uses: chrislennon/action-aws-cli@v1.1

      - name: Deploy to S3
        run: aws s3 sync ./public/ s3://${{ secrets.AWS_S3_BUCKET }} --acl public-read --follow-symlinks --delete --cache-control="max-age=3600"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

I have a website hosted at cbelsole.com where I can push content that I think would be useful for people. Total development time 1 day.

# Future goals

1. Checking all of my images to Github is eventually going to blow up the size of the repository. Eventually I want a workflow that syncs images with s3 and magically creates links.

2. Ironically, after looking more into [stackedit.io](https://stackedit.io/) it may be the CMS solution I was looking for.

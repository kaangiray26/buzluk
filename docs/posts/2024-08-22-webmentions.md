---
title: Using Webmentions with GitHub Actions
date: 2024-08-22
author: kaangiray26
tags: [indieweb, webmentions, jekyll, blog]
---

# {{ $frontmatter.title }}

:date: August 22, 2024

Testing webmentions for the first time. This post will send a webmention to [another post](https://www.buzl.uk/2024/08/21/pyright.html) on my blog. Let's see if it works.

**Edit:** It worked! I can see the webmention on my other post. Okay, let's go over how I set this up and how you can do it too.

## First, what are webmentions?

Webmention is another web standard recommendation from [W3C](https://www.w3.org/TR/webmention/), that was written by the [IndieWeb](https://indieweb.org/Webmention) community. Basically, it describes a way for website owners to notify each other when they link each other's content on their posts. With Webmention, you can leave comments, likes, reposts, and other interactions on other people's websites, especially if you use [microformats](https://indieweb.org/microformats) with it.

I first learned about it on [James' Blog](https://jamesg.blog/2024/02/19/personal-website-ideas/) and thought it was a pretty new thing. But it turns out, it's been around since 2016. I guess I'm late to the party.

There are two sides to webmentions: receiving and sending. Let's start with the easier one.

## Part 1: Receiving

Notifications are simple HTTP requests. For that reason, we need a server that listens for incoming pings. I'm using a popular choice called [Webmention.io](https://webmention.io/), created by [Aaron Parecki](https://aaronparecki.com/). It's one of those precious services that are free and open-source on the web. To set it up, you just create an account on the website, add your domain, and put a link tag on your website's head section. This is the one I have on my blog:

```html
<link rel="webmention" href="https://webmention.io/www.buzl.uk/webmention" />
```

With this tag, other websites can recognize that we accept webmentions. Showing them on your page is also very easy. I have a little script located at [webmentions.js](/assets/scripts/webmentions.js) that fetches the webmentions from the service based on the URL of the post you're viewing as the target. Then, it adds them to the page as a list and shows them in the DOM. If there are no webmentions, the webmentions section is hidden.

## Part 2: Sending

Sending webmentions manually is a bit tiresome. When you publish a post with links, for each link, you have to check if they support webmentions, and then send a request to their webmention endpoint containing the `source` and `target` URLs. Some people use services like [Webmention.app](https://webmention.app/) to automate this process, but I wanted to do it myself.

A little bit background about my blog: I use Jekyll as my static site generator, and I host it on GitHub Pages. For the build process, I just use the default GitHub Actions workflow that is recommended by Jekyll. With this setup, I just push some changes and the site is build and deployed automatically.

Now, with GitHub Actions, we can try to automate the webmention sending process. All we have to do is to collect all links from my posts, check the target URLs and find out if they support webmentions, and then send a request to their webmention endpoint. For this, we first take the `sitemap.xml` file that is generated by Jekyll and extract all the post URLs from it. These are the URLs of our blog posts. Then, we load each URL and extract all the links from the post.

However, we also need to save the webmentions we sent to avoid processing the same URL multiple times in future runs. For this, after the end of the workflow, we save the URLs as a JSON file as an asset by releasing a new release. We also save the domains that don't support webmentions to avoid checking them again. This way, we save the state of the workflow and don't have to check the same URLs again.

For the actual processing part, we have a Python script that does the job. I'm not going to go into the details but it just makes some HTTP requests and parses the HTML with [lxml](https://lxml.de/). It's not perfect but it does the job for now.

I've added the files to a new repository, which you can check out [here](https://github.com/kaangiray26/webmention) if you want to adapt it to your own blog. My blog's source code is also available [here](https://github.com/kaangiray26/buzl.uk) if you want to see how I implemented the parts I mentioned above and below.

If you want to use this setup, here are the steps you need to follow:

1. Create an account on Webmention.io and add your domain.
2. Add the link tag to your website's head section.
3. Have a `sitemap.xml` file on your website.
4. Add a `TOKEN` secret to your repository with your GitHub token.
5. Add the GitHub Actions [workflow](https://github.com/kaangiray26/webmention/blob/main/main.yml) to your repository. You don't have to change anything in it.
6. Deploy your website to trigger the workflow.

That's it! You should now be able to send webmentions automatically with GitHub Actions. If you have any questions, feel free to ask me, or send me a webmention. I'd love to see if it works for you too.

---
layout: post
title: "Moved to Azure Static Web Apps"
date: 2020-05-30T00:33:00-04:00
author: "Michael S. Collier"
description: Moving my blog to Azure Static Web Apps.
tags: [azure-static-web-apps]
comments: true
---

I've moved my blog again!  Well, sort of.  Not a new engine this time.  I've moved to a new host!

I've moved my blog to [Azure Static Web Apps](https://docs.microsoft.com/azure/static-web-apps/). The docs for Static Web Apps (SWA) include a good [tutorial on using Hugo with SWA](https://docs.microsoft.com/azure/static-web-apps/publish-hugo).  I followed that tutorial.

Once I got the content set up and GitHub action publishing working, it was time to set up the custom domain name.  Again, the Microsoft docs help there too.  I was able to follow the [instructions for setting up a custom domain with SWA](https://docs.microsoft.com/azure/static-web-apps/custom-domain), and then Burke Holland's post on using [CloudFlare for setting up a root domain](https://burkeholland.github.io/posts/static-app-root-domain/).

Overall, a pretty easy process.
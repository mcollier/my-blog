---
title: "Azure Functions - Team ASCII Art Wins"
date: 2020-11-17T08:15:00-05:00
draft: false
author: "Michael S. Collier"
tags: [azure-functions]
comments: true
---

A few months ago one of the greatest scandals to hit Azure Functions erupted . . . ASCIIartgate!

[![something](../../images/azure-functions-ascii-art-is-back/giphy.gif)](http://gph.is/1WDltOQ)

[#teamAsciiArt](https://twitter.com/hashtag/teamAsciiArt) became a trending topic on Twitter.  Maybe.

## The Big Deal

In an [effort](https://github.com/Azure/azure-functions-core-tools/issues/1131) to reduce the verbosity of logging output by `func start` in the Azure Functions Core Tools, the decision was made to remove the famed ASCII art.  To be fair, running `func start` did output quite a bit of logs.  The ASCII art version of the Azure Functions logo was just one of those logs.  It just happened to be the most famous of those logs.  The logo was the thing of highly coveted t-shirts!

Thus, starting with Azure Functions Core Tools version [3.0.2881](https://github.com/Azure/azure-functions-core-tools/releases/tag/3.0.2881) in September 2020, the logging got a lot cleaner.  There are fewer noisy debugging statements when executing `func start`.  The startup _feels_ snappier too.  That's goodness.  However, the Azure Functions ASCII art logo was no more!

_(2020 . . . it just keeps giving)_

## It's Baaaaaaaack

Thanks to millions and millions of fans (right?) and a very responsive and fantastic set of Azure Functions PMs and engineers, the Azure Functions ASCII art logo is back!  Sort of.  

Azure Functions Core Tools version [3.0.2996](https://github.com/Azure/azure-functions-core-tools/releases/tag/3.0.2996) brings back the ASCII art.  You will need to set an environment variable to do so though.  By default, `func start` still looks like this:

![Azure Functions Core Tools with no ASCII art logo](../../images/azure-functions-ascii-art-is-back/func-core-tools-no-ascii.png)

Setting the FUNCTIONS_CORE_TOOLS_DISPLAY_LOGO environment variable to `true` will bring you ASCII art joy!

![Azure Functions Core Tools with ASCII art logo](../../images/azure-functions-ascii-art-is-back/func-core-tools-ascii.png)
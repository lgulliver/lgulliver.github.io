---
author: Liam Gulliver
title: Azureish Live! Building a Git Commit Watcher with Azure Functions and GitHub [Part 1]
tags: devops azure github serverless twitch youtube liam-gulliver pete-gallagher jonathan-relf azure-functions webhooks signalr dotnet dotnet-core blazor
date: 2020-10-28 23:30:00
header:
  video:
    id: H4dQ87hyzY8
    provider: youtube
---

This week I, along with my friends [Pete Gallagher](https://twitter.com/pete_codes) and [Jonathan Relf](https://twitter.com/jbjon) launched a new show dedicated to live coding with Azure called [Azureish Live!](https://twitch.tv/azureishlive).

The idea behind the show is that for all the projects we come up with and build, we show the whole process on the show. From conception to deployment, using Azure resources. Think of it a little like the show [Scrapheap Challenge/Junkyard Wars](https://www.imdb.com/title/tt2007689/) of a few years ago, except we aren't competing against anyone and working together to help teach ourselves something new and the audience. 

We'll also be pushing our code to public repositories on our [GitHub](https://github.com/AzureishLive) as we go along too. Audience participation is encouraged so feel free to open issues and pull requests as we go too.

It's live on [Twitch](https://twitch.tv/azureishlive) every other Tuesday starting Octber 27th, 2020 at 12:30PM UK time for 1 hour and is available on our [YouTube channel](https://www.youtube.com/channel/UCVQtNIXAgtJA-w9pd17WH5A) shortly after for on-demand.

The show is something we've been figuring out in the background for a little bit and we thought it'd be fun to do. We've had a blast on our first show so please consider subscribing to both our [Twitch](https://twitch.tv/azureishlive) and [YouTube](https://www.youtube.com/channel/UCVQtNIXAgtJA-w9pd17WH5A) channels.

In our first show, we start the journey of building our first project: a git commit watcher for the show itself! To do it, we've decided to use Azure Functions to consume webhooks from Github and eventually update a Blazor UI using SignalR to display the latest commit on our stream.

Check out the repository over on [GitHub](https://github.com/AzureishLive/gitwatcher)!

We didn't get all the way through it in the show on this occasion but we did manage to design our solution and start putting together some of the pieces of the puzzle get moving.
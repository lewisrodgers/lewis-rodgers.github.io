---
layout: post
title:  "Using Postman to make calls to Google APIs for non-developers"
category: Development
tags: []
---

There are a few ways to make API calls to Google services like Calendar, Drive, and the user directory. One of the easiest ways is with the [Google APIs Explorer](https://developers.google.com/apis-explorer). It's a great tool to make quick adhoc request for inspect the returned data. 

And, while the explorer keeps a history of executed requests, it only persists during the current session. In other words, reloading the page clears the history. Plus, you're unable to adjust the parameters that were used to make the original request. 

That's not ideal when you're working through a solution, because it may take a few iterations before it's solved. You'll want to "replay" requests and make adjustments here and there.

Another way is to write code. But, while you're technical enough to make API requests, programming isn't your thing. And you certainly don't have time to set up a development environment.

Then there's a tool like Postman. There's some setup involved, but once you get going it's by far the best way to go.
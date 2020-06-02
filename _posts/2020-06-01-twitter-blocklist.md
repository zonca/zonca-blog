---
layout: post
title: Import and export list of blocked users on Twitter
categories: [python, twitter]
---

[Back in 2015](https://blog.twitter.com/en_us/a/2015/sharing-block-lists-to-help-make-twitter-safer.html), Twitter
provided the feature to export and import lists of blocked users, unfortunately they discontinued this
service.

In order to fill this void, <https://blocktogether.org/> provides a great service to allow users to subscribe
to public block-lists that are always kept updated by some maintainers,
unfortunately they have capacity issues and needed to impose limits on number of subscribers and list length.

I have written a small Python script that can provide a self-service import and export functionality
using the Twitter API.

See <https://github.com/zonca/twitter_blocklist>

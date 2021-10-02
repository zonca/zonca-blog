---
layout: post
title: Monitor traffic on Github repositories
categories: [github]
---

Github only shows traffic data for the last 2 weeks.
Through the API is possible to gather those data every 2 weeks and save them for later collection and reduction. 

## Available data

It's 3 different stats:

* clone: number of clones and unique clones per day
* referrer: websites that linked to the repository in the last 2 weeks
* traffic: views and unique visitors

## Scripts

I have created a set of scripts based on [`github-traffic-stats`](https://github.com/nchah/github-traffic-stats)

Follow the instructions in the `README.md`:

* <https://github.com/zonca/save-github-traffic-stats>

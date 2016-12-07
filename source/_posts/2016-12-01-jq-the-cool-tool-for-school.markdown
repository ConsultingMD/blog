---
layout: post
title: "jq - The Cool Tool For School"
date: 2016-12-01 08:00:17 -0700
comments: true
author: "Wen Li <wen.li@grandrounds.com>"
categories: tooling, cli
---
There's something incredibly satisfying about finding the perfect tool for a job. If you deal with JSON and you haven’t played with jq, you’re in for a real treat.


The description from their [site](https://stedolan.github.io/jq/) states, 

> “jq is like sed for JSON data - you can use it to slice and filter and map and transform structured data with the same ease that sed, awk, grep and friends let you play with text… …jq can mangle the data format that you have into the one that you want with very little effort, and the program to do so is often shorter and simpler than you’d expect.”


I’ll go over a few use cases I’ve had for it so far.

### Simple Case
The most basic and neat thing you can do with jq is to pipe JSON into jq to pretty-print it using `'.'` 
Whether the JSON is from a curled response:

`curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5' | jq '.' `

or just some JSON you need to use:

`cat template.json | jq ‘.’`

<img src='http://i.imgur.com/XG4JwOp.png' text-align='center' width="400px">

It's pretty and readable! This becomes more useful when you're dealing with messier JSON.

### Validation
I’ve also used it as a way to find errors in what I assumed was valid JSON format. My use case has been for when DynamoDB expects imports in a special JSON format. I found out that the import failed, but not much more than that. Knowing that the most likely culprit is probably illegitimate JSON somewhere in the mountains of data, we piped it all through jq to find an exact line of where an extra quotation had creeped into one file. Talk about a needle in the haystack.

This is also especially useful when you want to validate JSON and the data could be sensitive information.

### Sorting Keys
Have you ever had to diff a large piece of JSON with another JSON only to find out that the keys are in a different order? Your tooling tells you that all of the things are not like the others which ends up not being useful at all. 


My generated CloudFormation template keys were in a completely different order than GetTemplate from the current stack, and I needed a way tell what the delta between the two was. The `-S` or `--sort-keys` option will sort the keys of your hash all the way down so that two sorted equivalent hashes will be identical. Knowing that, I was able to create two sorted files and diff them from there.

`cat actual.json | jq -S . > asorted.json`

`cat proposed.json | jq -S. > bsorted.json`

Use your favorite diffing tool and the problem resolved in three lines!
`meld asorted.json bsorted.json`


### More Info
There are other neat features to jq, such as very powerful filtering. You can find out more in their manual here: (https://stedolan.github.io/jq/manual/)

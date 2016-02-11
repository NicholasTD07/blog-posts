---
title: "How to Extend Argo with Custom Type Parser - TL;DR"
created_at: 2015-06-06 20:00:00
kind: article

tags:
  - Swift
  - Swift 1.2

permalink: "/tldr/2015/06/07/tldr-how-to-extend-argo-with-custom-type-parser.html"
---

Example code of converting strings to different types, e.g. `NSURL`, `NSDate`, during JSON decoding using [Argo](https://github.com/thoughtbot/Argo), the functional JSON decoding framework written in Swift.

<!-- more -->

<!-- TODO -->
One of the [TL;DR]({{ site.url }}/tldr/) series.

It shows how to extend [Argo](https://github.com/thoughtbot/Argo)'s ability to parse different types of data, e.g. `NSURL`, `NSDate`, and/or `Int` in a `String`.

If you want to play with the following code, you can copy and paste it into [Argo](https://github.com/thoughtbot/Argo)'s playground.

<script src="https://gist.github.com/NicholasTD07/67fb7a96b93d6ca828bf.js"></script>

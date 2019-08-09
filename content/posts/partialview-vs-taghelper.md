---
title: "PartialView vs TagHelper : A Performance Comparison"
date: 2018-11-01T10:49:41+03:30
draft: false
---

TagHelper is a new feature in ASP.NET Core. We usually use TagHelpers to generate HTML like tags and compared to the old HtmlHelpers, TagHelpers are a much cleaner approach.

In many cases we can use TagHelpers to replace PartialViews too. of course developing TagHelpers are a lot more difficult and need to put on more time.

In this experiment I try to compare using PartialViews vs TagHelpers performance wise. I'm not sure if this is a totally legitimate way to compare in theory, But in practice the code here is what web developer use in a daily basis. So it can give you some ideas when and where to use which.

# TL;DR

Seems like TagHelpers are about **5x** faster compared to same PartialView.
![PartialView vs TagHelper](https://github.com/codehaks/codehaks.github.io/raw/master/img/post/partialviewperfchart.png "PartialView vs TagHelper")

## The situation

For one of my projects I used a **PartialView** to show star rating And I have the same functionality using a **TagHelper**. I want to compare these two performance wise and which one is faster.

### PartialView

<script src="https://gist.github.com/codehaks/137d74cd91604bf7985e45be6e851e86.js"></script>

### TagHelper

<script src="https://gist.github.com/codehaks/5894796c4f47b665c656ac8d9a62b054.js"></script>

### Generated HTML

<script src="https://gist.github.com/codehaks/16367e9c1a91fdeb161e622856f5ae64.js"></script>

and the result is something like this :
![star Rating TagHelper](https://github.com/codehaks/codehaks.github.io/raw/master/img/post/samples-stars.png "Star rating TagHelper")

## The Experiment

To test the performance between the two, I created a new Empty ASP.NET Core project and added the MVC service and Middleware. I used RazorPages to add two different views. In each view I have a **for-loop** which runs for **50,000** times and prints out the stars from rating 1 to 5.I measure the elapsed time using Diagnostics's Stop Watch.

### Using PartialView

<script src="https://gist.github.com/codehaks/2ec59695f3424fc55e0b5529337cbfd5.js"></script>

### Using TagHelper

<script src="https://gist.github.com/codehaks/eb3593d1094334f54501865950421fba.js"></script>

Next I ran both pages for 10 times each and every time the TagHelper version seem to be a lot faster. I used Fire Fox Developer Edition but the result are much the same in other browsers.

## Conclusion

On average a TagHelper runs at least 5x faster than the same PartialView. Also the result is independent from using static files or not. you dont need to show the actual stars to see the performance advantage.

## Download source code

You can download the full project and test it yourself here :
[PartialViewPerfTest project](https://github.com/codehaks/PartialViewPerfTest)
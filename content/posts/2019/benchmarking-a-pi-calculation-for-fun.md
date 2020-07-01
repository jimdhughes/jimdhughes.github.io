+++
author = "James D Hughes"
categories = ["nodejs", "golang", "java", "python", "c", "programming", "pi", "algorithms"]
date = 2019-05-22T09:51:39Z
description = ""
draft = false
image = "/images/2019/05/DSC_1114.JPG"
slug = "benchmarking-a-pi-calculation-for-fun"
tags = ["nodejs", "golang", "java", "python", "c", "programming", "pi", "algorithms"]
title = "Benchmarking a pi calculation for fun"

+++


Dotnet Core, Golang, Node, C, Java, Python.

This is what I felt like doing on a Tuesday night after seeing yet another article discussing the performance of dotnet core over node.js.  As is typical when things like these arise, our team's slack channel went into the lightest of debates about C# vs Node, compiled speeds vs interpreted languages, and finally back to what we were originally discussing - "Do you think cypress would be faster than selenium?"

An article that came up in slack is here: [https://www.skylinetechnologies.com/Blog/Skyline-Blog/March_2018/pi-day-compare-dotnet-core-classic-node-javascript](https://www.skylinetechnologies.com/Blog/Skyline-Blog/March_2018/pi-day-compare-dotnet-core-classic-node-javascript)

Which was neat because it was celebrating pi day by using dotnetcore, dotnet classic, and node to calculate pi using the [Leibniz formula for π](https://en.wikipedia.org/wiki/Leibniz_formula_for_%CF%80) and I like geeky things like that. This post isn't about what that is - but if you're interested visit that link and check it out!

> Who was the largest knight at King Arthur's Round table? Sir Cumference!
> He got that way from eating too much pi!
> -- Unknown

The point of the article was to showcase how much faster dotnetcore was than nodejs. Which it seemed to.. until I looked at their github and noticed that the code wasn't necessarily comparing apples to apples.

In dotnetcore they used the standard library and for nodejs they installed and included [mathjs](https://mathjs.org/) [] and [moment](https://momentjs.com/) for performing some small math and date functions.  Libraries like this are both the boon and bane of node.js.  It was completely possible to use JavaScript's native Math and Date api however, for ease of use, the authors chose to install these two libraries. They make programming in node _easy_ while sacrificing some performance - a great tradeoff in a lot of circumstances!

However, I digress. Long story short - using node v10.6 and refactoring the libraries back to the standard library, I was able to bring the node performance from ~9 seconds in windows to ~4 seconds which isn't as good as dotnetcore on windows but is much faster than original.

This led me to wonder - how would other languages perform?  Golang has been a personal favourite of mine for a while despite not having anything in any intense form of production using it - so I started there and was ... disappointed.

It was about 10 seconds - so I did what any rational programmer with 2 small kids would do after they go to bed. I tried the other languages I know how to program in!

|Language | Version | Time (ms)
|---|---|---|
|Golang | 1.11.2 | 10 852
|Node | 12.3 | 897 
| C | C11 |867
| Python | 2.7.15 | 25 704
| Dotnet Core (C#) | 2.2.0 | 1 231
| Java | 1.8.0_818 |7 816

My golang sweetheart fell by the wayside for this test :(

My upgraded version of Node however killed it! I originally ran it with node v10.6 and it was ~ 3.4 seconds then upgraded node just to see. Now I don't feel like downgrading node so I'll leave that up to whoever happens to stumble across this to play with.

I'm also not going to try it on many vm's because I'm truthfully much to lazy for that this evening however if you are so interested, I ran these on OSX 10.

I just wanted to share how differently different languages handle the same sort of calculations. I'm curious to know that if I used Oracle's Java vs OpenJDK's Java if the performance would have increased?

That's about all I have to say about this so now I do as my young ones have done - and hit the hay!


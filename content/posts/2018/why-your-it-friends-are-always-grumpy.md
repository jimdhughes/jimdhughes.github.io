+++
author = "James D Hughes"
date = 2018-04-21T01:28:28Z
description = ""
draft = false
slug = "why-your-it-friends-are-always-grumpy"
title = "Why your IT friends are always grumpy"

+++


Humor me for a second here.

It might just be my company - but I doubt it.
Watching VLOGs you see lots of developers that have some common traits:
1. Tired all the time
2. Sarcastic AF
3. Refusal to help you fix your PC at family dinners

The meme's are endless.
![It Works!](https://i.imgur.com/RHpy3Ho.jpg)

Note: Google Images stopped letting me follow links here so I'll have to revisit it. As if to prove the point of my post.

![Cant break it!](https://i.imgflip.com/1wnc14.jpg)
![Errors](https://i.imgur.com/YtVYBPU.png)

There are several reasons for this:
1. They stayed up until 2:00 am last night fixing the bug they introduced that caused your fancy new timesheet system to suddenly sell cars.
2. They don't remember how to speak as their entire interaction comes from memes while trying to stifle their 2:00 am tears.
3. Just because they write software doesn't mean they have extended knowledge of how laptop components are assembled.
  3.1 Also, your macbook uses proprietary Mac parts so je refuse.

Fundamentally though, I think the reason that most IT folks, inclusive of those in Infrastructure and Software, are grumpy are because they have been conditioned to be so.


Keep following me.

My undergrad software development courses DRILLED into my head: TDD.
Test Driven Development.  If you even read this then I'm sure you know what I'm talking about. If you don't it pretty much goes like this:

Step 1: Write tests that will fail
Step 2: Write code to pass those tests
IF your code passes all the tests any of the following may be true:
 - Your tests were garbage. Write better tests
 - You didn't consider edge case 1-1 billion.
 - You can't divide by 0.

We've been conditioned to think of all the possible ways that code WILL fail.  The same idea goes for your infrastructure crews.  They suddenly have to support some new technology because some executive wants it for some reason.
Question 1: What business problem will this solve?
Question 2: Why can't excel just do it?
Question 3: How can we run this on I.E 6?

No matter what though, you're now running a blockchain application on your network with 3 arbitrary VM's set up because apparently that makes sense to your executive team.  Your consensus may as well be a proof of authority algorithm (i.e do what I said cuz I said it) as your network is owned solely by you.  Now you've got 10 distributed asset transferring apps from developer tutorials that will never be used in production but is on the corporate resume of 'We do blockchain'. 

![do you need a blockchain?](https://cdn-images-1.medium.com/max/800/0*850OATxY25S23TGR.jpg)
Note: A plain old webapp could have sufficed for our previous explanation. As without their being stakes held by multiple groups of interest, there's literally no need for a blockchain, IMO.  I got excited by it when it came out first. I even had my next project planned out using blockchain as my back end.  Then I realized that putting a bunch of nodes on a network owned by one entity so the entity could verify itself made no sense whatsoever.  Below is a less sarcastic flowchart for if you actually need a blockchain.
![blockchain flowchart](https://cdn-images-1.medium.com/max/1600/1*MK-TTVmz6k6-7uTD8iV73g.jpeg)

If you haven't gotten grumpy by this, then I'll explain why they are:
* The mandate is by people who don't understand what a blockchain is.
* There's no budget to upgrade existing apps so they can run on a modern browser, but we have pointless blockchain infrastructure.
* No one considered the implications of setting up a distributed network of linux VM's in a department that is entirely windows based and now we have a ransomware outbreak.

Something can *always* go wrong. It's the job of your IT groups to either prevent things from going wrong or to prepare to salvage everything if something does go wrong.

Now that I'm done ranting about the blockchain, I'd like to contiue talking about why this predisposition to 'Something can always go wrong' is *probably* terrible for the mental well being of your tech guys.

##1. Talk about a bummer.
It's 2018 and discussions around mental health is huge.  We've got so many issues to pay attention to and the access to information is through the roof.  News outlets are pounding us with the most dramatic of events - both happy and sad. Now here your tech guys are, reading all of this and at the same time trying to account for every possible failure point in their codes design.
It reminds me of the book "The Subtle Art of Not Giving a F\*ck" (a fantastic read btw)

> “Life is essentially an endless series of problems. The solution to one problem is merely the creation of another.” 
― Mark Manson, The Subtle Art of Not Giving a F*ck: A Counterintuitive Approach to Living a Good Life

Could you imagine a life where you're spending a disproportionate amount of your time determining how your work is going to fail? Perhaps coming up with new and innovative ideas of making it fail? That's what the ladies and fellows building all life's conveniences do every day!

##2. Negativity breeds negativity
Without the right attitude, your developers take this consistent negative view of the world and may become *extremely* resistent to trying anything new.  With new comes unknowns and with unknowns come risks and when the fate of a business rests on your ability to create flawless code, risks are **bad**.  Spiralling in this vortex of fear you get a bunch of reclusive meming grumps wondering why they chose their path.

I saw a question on [Quora](https://www.quora.com/) today and it is really what sparked me to write about this.  "What happens to older programmers? Do they get fired as they get older and 'less innovative'? ..."
This resonated a bit since I'm a developer and I'm also in my 30s.

With resistance to experimentation, people can stagnate.  Technology is moving at a clip that is just impossible to fathom and it's really easy to make a wrong career decision and become a specialist in technology *x* as opposed to *y* and all of a sudden become irrelevant.

For that reason, this has really become a double edged sword. On one edge, experimentation can open you up to new risks and leave you vulerable for more failure. On the other, without exposure you fade away into deprecation.

Now, not all is lost! There are upsides to this (in my opinion at least)

Understanding what and why is a big part on your road to happiness.  Realizing that you've been conditioned to account for all the bad things that chould possibly happen could help you open your mind back up to exploring the unknown and accepting some risk.  Think back to your naive 20's where you were "innovative" and "game changing" or the like.  You were too dumb to realize that what you were attempting was a massive security vulnerability.  It's our knowledge and experience that hamstrings us.

I write all of this from a proverbial position of 'privilige' in that at my current employ, I'm capable of taking risks and trying new things. I have apps that were written in  Java with Swing GUI's, some GWT (Google Web Toolkit) apps, a few MEAN  apps, one or two golang microservices, and at least 1 Ruby on Rails app.  I get to play with the 'flavours of the week' and can throw together some apps using SQL or NoSQL technologies.

Most developers don't have this opportunity though and I could see how they could spiral into a grumpy disarray.

Long story short - If you're a developer, take this into account! Think about how you can spice up your innovation, open your mind a bit and maybe bring some new value added ideas to your company.

If you're a manager of developers, or an external stakeholder, consider this in your strategies to spark innovation.  Send them off to a conference about an emerging technology they're interested in.  Give them the same speal that you give everyone else...
#It's OK to Fail
Startups are innovative because it's ok to fail.
Companies are stifiling because you simply cannot.

Understanding that jobs need to get done and the like, I'd like to toss up a possible solution:
Open up your R&D groups to your regulars. If you don't have an R&D group, start one.  Let your bored team start building out proof of concepts in a controlled environment.  An afternoon a week? A day a week? Dedicated to trying someting *new* and discoviering new business value.

I'll end this rant here as I don't think I'm being helpful anymore.  I hope I gave you something to think about today.


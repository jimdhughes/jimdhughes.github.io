+++
author = "James D Hughes"
date = 2020-01-01T00:49:30Z
description = ""
draft = false
slug = "nasa-has-an-api"
title = "NASA has an API!"

+++


This started as an article on me learning Vue.js

As I was going through the motions of the annoying and overdone TODO application to learn a JS library, I decided to do a quick scour of the internet for a public API that might jazz things up a bit.  After finding a few cat and meme generators, I stumbled across this gem!

[https://api.nasa.gov/](https://api.nasa.gov/)

That's right - NASA has a public API.

There are limited calls that you get for free and then you have to register for an account to get bumped up to (a thousand) requests per hour. I found myself quickly over-excited and burning through all my free api calls so I registered for a key.  This post is going to be a little less about 'building an app with Vue' and more about the journey that took my 'evening learning' journey to a _slightly_ over-designed monster of an app.

# The Journey

## The naive programmer

I started with all the easy stuff.  Install Vue.js and make a web service that calls api.nasa.gov directly.  I decided to start with their APOD api as it was the most sexy one available: https://api.nasa.gov/planetary/apod

With all the awesome things that vue and other tooling these days provide with automatic refresh / reload of your website on code change,  I very quickly ran out of free calls and got throttled. So I proceeded to register for an API key when I almost committed a cardinal sin of programming.

> Thou shalt not store api keys in your source code

This should be a no-brainer.  In case yo didn't know, there are tools that exist specifically to parse through old git commits to extract keys that _might_ have been in there at some point. ([https://www.zdnet.com/article/trufflehog-high-entropy-key-hunter-released-to-the-masses/](https://www.zdnet.com/article/trufflehog-high-entropy-key-hunter-released-to-the-masses/)) It's a bad thing to do. Don't do it. This led me to a new level of complexity to my tiny app - I now need a back end service to proxy requests through to the NASA API. This is where my API keys will be stored and is not available to the general public, nor my source control ;)

## The Web Proxy

For reasons beyond my explanation, I always decide to build these little things using golang.  I do it because I really want to build a production grade app using it and I just don't have the bandwidth to do it. My work is a heavy .NET shop (.NET core at least now) so I don't get to play with my fun emerging technologies all that often.

I started building the app using an embedded database `etcd-io/bbolt` ([https://github.com/etcd-io/bbolt](https://github.com/etcd-io/bbolt)) because I really wanted to keep this app as simple as humanly possible.  This was specifically for caching API responses. Even with the new heavier limit on my API keys, I figured that it is pointless burning API calls for data that I've already retrieved.  Once I got the app rolling along and I was caching data in bbolt and browsing it with `boltbrowser`, I thought of something I hadn't before ..

> Insert Grinch GIF here

I should _probably_ respect NASA's data and not store it on my machine indefinitely. Thus began the next round of research.  At this point I had a few options:

1. Find a way to set a TTL on my cached objects and expire them once TTL was reached
2. Delete the database daily and thereby only retaining at most 24 hours worth of data
3. Find a better mouse trap - or in this case, database solution.

There are pros and cons to all of these...

### 1.  Find a way to set a TTL on my cached objects and expire them once TTL was reached

Pros:

* No rewrite of the persistence layer
* Add functionality to set TTL keys on save
* Add functionality to delete keys on TTL

Cons:

* More code to write and maintain

### 2. Delete the database daily and thereby only retaining at most 24 hours worth of data

Pros:

* It's ... easy?

Cons:

* Now I need a cron job or something on the OS to clear it out (not very CI/CD friendly, not contained)

### 3. Find a better mouse trap - or in this case, database solution.

Pros:

* Redis has self-expiring keys
* No more code to write to handle expiration

Cons:

* We need to rewrite our persistence layer

Ultimately I decided that I wanted to not maintain a lot of code so I decided to just strip out my old persistence layer and update it to be compatible with Redis. I've added a dependency to my solution but it provided a lot of functionality I otherwise would have had to write.  It's an age old battle.

## More Refactors!

With all the time I saved switching my DB solution from `bbolt` to `Redis` I decided I may as well extract my api wrapper into a separate library. I figured that, one day, I might have more time / be more interested in calling more NASA endpoints and it would be much easier for people to help out, if they were so interested, committing to the library repository as opposed to my pet project repository. So off to the races I went!

After a bit of refactoring and pumping up into git, I had a library that I was ready to consume because of go's slick depency model - that is, cloning and including go files from git repositories! I created the proxy at https://github.com/jimdhughes/nasa and was able to include it simply by runing a `go get` and then importing it in my `handlers.go` file in my `nasa-proxy-api` project with a simple

```golang
import (
	"encoding/json"
	"errors"
	"log"
	"net/http"
	"time"

	"github.com/jimdhughes/nasa"
	"github.com/jimdhughes/nasa/models"
	"github.com/gorilla/mux"
)
```

It was around this time where my wife started making fun of me for being a turbo nerd so I decided to call it the quits after implementing only the endpoints I was truly interested in:

* APOD
* Mars Weather (my data model is different than NASA's because I relaly hated NASA's.  Truthfully, this could use a refactor to serialize a bit prettier honestly)
* Near Earth Objects
* Image Video Library

## Main takeaways

### Don't be afraid to be wrong

One of my biggest faults as a person is my fear of being wrong.  Maybe not my fear, but my disgust with myself for being wrong.  Call it nuts but I can't think of anyone that goes forward with an attitude of "I'm wrong all the time and I love it!" - it's okay to look back at work you've done and realize that you could have done it better and/or differently.  In fact, if you haven't done that ever in your career then you're either a liar or you're a god.  It's okay to be wrong as long as your original decision was justified given the information you had at that time.

### Code cleanly

I'm not the cleanest coder but I try to keep things pretty delineated. In this example, I only had to refactor one small portion of my application to swap out the `bbolt` vs `redis` persistence layer. Granted that it is a super small app, it would have still been a headache if my persistence was built in tightly with my routers.  Coding cleanly allows you to make these refactors when either you realize that you messed up immensely or new technology was introduced which will make your life easier.

### Have fun!

I've had so many projects stop before they started because I tried prematurely optimizing.  Spending a week on a data model for an application that may or may not ever get off the ground is not conducive to starting a company. Work fast, be flexible, and code well. All of this will let you keep having fun as you progress in your career building new apps, rewriting legacy apps, or kicking off your startup :)

# Repos

Here are the git repos for these projects if you want to mess around.

<figure>
	     <a href="https://github.com/jimdhughes/nasa-proxy-api">
	       <div>
	         <div>Daimonos/nasa-proxy-api</div>
	         <div>Proxy APII for requesting and caching requests. Contribute to Daimonos/nasa-proxy-api development by creating an account on GitHub.</div>
	         <div>
	           <img src="https://github.githubassets.com/favicon.ico">
	           <span>Daimonos</span>
	           <span>GitHub</span>
	         </div>
	       </div>
	       <div><img src="https://avatars0.githubusercontent.com/u/3279480?s=400&v=4"></div>
	     </a>
	     <figcaption>API Proxy</figcaption>
	   </figure>

<figure>
	     <a href="https://github.com/jimdhughes/nasa">
	       <div>
	         <div>Daimonos/nasa</div>
	         <div>Contribute to Daimonos/nasa development by creating an account on GitHub.</div>
	         <div>
	           <img src="https://github.githubassets.com/favicon.ico">
	           <span>Daimonos</span>
	           <span>GitHub</span>
	         </div>
	       </div>
	       <div><img src="https://avatars0.githubusercontent.com/u/3279480?s=400&v=4"></div>
	     </a>
	     <figcaption>Golang Library</figcaption>
	   </figure>

<figure>
	     <a href="https://github.com/jimdhughes/nasa-browser">
	       <div>
	         <div>Daimonos/nasa-browser</div>
	         <div>A Vue.js app for browsing the nasa apis. Contribute to Daimonos/nasa-browser development by creating an account on GitHub.</div>
	         <div>
	           <img src="https://github.githubassets.com/favicon.ico">
	           <span>Daimonos</span>
	           <span>GitHub</span>
	         </div>
	       </div>
	       <div><img src="https://avatars0.githubusercontent.com/u/3279480?s=400&v=4"></div>
	     </a>
	     <figcaption>Vue.js client</figcaption>
	   </figure>






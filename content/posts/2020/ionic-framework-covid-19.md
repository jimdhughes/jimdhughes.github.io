+++
author = "James D Hughes"
date = 2020-04-04T10:35:39Z
description = ""
draft = true
slug = "ionic-framework-covid-19"
title = "Ionic Framework & COVID-19"
+++

That's a fairly cryptic title, isn't it.

Hopefully the folks at Ionic don't take this as a negative marketing slight as I'm really trying to be positive with this post. I'm here to help!

It should really be called 'FIND ME THE TP!' - Toilet Paper, not Town Portal, or Teleport, or any other abbreviation of TP. Anyways, I'm already far off topic.

I don't get to cut a lot of code anymore at work.

Scratch that. I don't get to cut any code anymore. At least officially sanctioned code. When I'm done sitting in my basement for self-isolation IT work, I decide that I'm going to sit in my basement for more self-isolation IT work that's much less profitable. In fact, some may say that it's entirely not profitable. I have a few friends that live by the addage that if you're working ant not making money then you're technically losing money. This is a fairly sage saying with deep seeded roots in the idea that you should always know your worth however that isn't entirely what this blog is about.

I was browsing the web looking for something to tackle. I see a lot of COVID-19 related apps that are showing similar things - infections in my area, deaths, and other such depressing facts. Other depressing facts are that we are in a crisis for TP, disinfectant wipes, and other life-essential items and I'm starting to run desperately low. Therefore, I thought that I may do my part to help with a civic service and create an app where people can share photos of TP if they happen to stumble across it and share it with the world! Or Country, Province, City, Town, Neighbourhood (if there are American readers then you're likely confused by my Province and ou in Neighbourhood but that's ok. We are different up here)

I'm going to mess around with Ionic Framework today. I'm intending on making a pretty simple app that has two functions:

1. Find TP near me (based off of community results)
2. Post a TP location - based on snapping a photo on your camera and using your phone's GPS to find you.

So let's just dive in.

I'm going to start on the front end code because, well, it shows results. Let's be honest - it's quite possible that I'll get half-way through implementing this and decide that I'm bored and move on to something else which happens with a great many of my 'Wife is at work on a Friday night and I'm bored AF' ideas.

### Scaffolding the app

I'm going to scaffold ionic app. I'll use angular and a tab layout because I'm lazy.

```
ionic start com.ineedtp.app --type=angular tabs
```

I'll add capacitor since I'm going to put it on a phone. Press `y` when that comes up.

This next part takes a few minutes - so I decided that I'm going to mix myself a beverage while I wait. If you want the full experience of the post, I suggest you do as well!

I'll start by doing some not-that-fun stuff and just renaming the tabs that will be found in `src/app/tabs/tabs.page.html` so that they will show some more relevant tabs names

```html
<ion-tabs>
  <ion-tab-bar slot="bottom">
    <ion-tab-button tab="tab1">
      <ion-icon name="search"></ion-icon>
      <ion-label>Find</ion-label>
    </ion-tab-button>

    <ion-tab-button tab="tab2">
      <ion-icon name="camera"></ion-icon>
      <ion-label>Share</ion-label>
    </ion-tab-button>

    <ion-tab-button tab="tab3">
      <ion-icon name="settings"></ion-icon>
      <ion-label>Settings</ion-label>
    </ion-tab-button>
  </ion-tab-bar>
</ion-tabs>
```

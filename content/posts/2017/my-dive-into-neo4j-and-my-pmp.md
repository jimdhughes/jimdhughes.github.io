+++
author = "James D Hughes"
categories = ["neo4j", "PMI", "PMBoK", "PMP", "GraphDB", "DataPorn", "Data"]
date = 2017-05-26T01:27:28Z
description = ""
draft = false
image = "/images/2018/05/Project-Risk-Management-Graph.png"
slug = "my-dive-into-neo4j-and-my-pmp"
tags = ["neo4j", "PMI", "PMBoK", "PMP", "GraphDB", "DataPorn", "Data"]
title = "My Dive into Neo4j - and my PMP"

+++


Man,
I need to get my life figured out!

I spend so much of my free time diving into new technology that I'm just waiting for my wife to pull the pin.  Sorry dearest! The nerdom never dies!

I just finished my application for my PMP exam (Project Management Professional - shameless self plug) and it was accepted. So once I scrounge up some dough I'll be paying to take the test and then we are off to the races! Goal reached.

I enjoy writing code, but I more enjoy envisioning systems that could be.  Since I was taking a prep course on project management, I decided to go ahead and take their layout of the information and turn it into (what it appeared to me to be) a graph.
I tried using [OrientDB](http://orientdb.com/) but it just felt klunky and weird and I wasn't digging it. So I dove into [Neo4j](https://neo4j.com/) and oh boy was it snazzy!

It let me make csv files and then import them and assign graphical allocations. It made transcribing the data pretty easy (a few hours of just typing into excel) then loading it up. Anyways, here's what the whole PMBoK looks like when you just use the cypher:
```language-cypher
MATCH (n) return n
```

![PMBoK Graph](https://www.dropbox.com/s/l844x8jtci7926a/PMBoK%20Graph.png?raw=1)

Frigging daunting right?
That's why PM's need to make a load of cash!

When you break it down into smaller chunks though, it's not that bad.
Here's what Project Risk Management looks like:

```language-cypher
MATCH (k:KnowledgeArea {name:'Project Risk Management'})<-[:BELONGS_TO]-(p:Process)-[:OUTPUTS|:INPUTS_TO|:UTILIZES]-(a) return k,p,a
```

![Project Risk Management](https://www.dropbox.com/s/h1r0gktnmha67tz/Project%20Risk%20Management%20Graph.png?raw=1)

I was thinking of how I could make this useful for future me. So I thought of a scenario: What happens if my Project charter changes during project execution? How many processes would that affect?

Upon taking a peek using:
```language-cypher
MATCH (e:Element {name:'Project charter'})-[:INPUTS_TO]->(p:Process) return e, p
```
![Project Charter Inputs To](https://www.dropbox.com/s/iisn6rra8dzsgqo/Project%20Charter%20Inputs.png?raw=1)

Not terrible I suppose.  The Project charter only inputs to 8 / 47 Processes.
But what about the downstream effects?

I'm not that great at this Neo4j game yet, but I did a couple queries to check what's up..
```language-cypher
MATCH (e:Element {name:'Project charter'})-[:INPUTS_TO]->(p:Process)-[:OUTPUTS]->(e1:Element) return e, p, e1
```

![Project Charter Inputs,  Process, Outputs](https://www.dropbox.com/s/no7t5yccxrdi69k/Project%20Charter%20Inputs%20-%20Process%20-%20Outputs.png?raw=1)

Look at this already!
The inter-dependencies being created by diving in just one more layer. We have some process outputs feeding inputs that were affected by existing inputs! Mind - Blown.

Project Charter -> Plan Scope Management -> Requirements Management Plan -> Collect Requirements

I'm only going to dive one level further since I think this is getting pretty clear now. However let's take a peek at what these newly affected elements feed into down the line shall we?

![More Processes And Elements And Processes](https://www.dropbox.com/s/331ixcxh4ex1oo7/Project%20Charter%20Inputs%20-%20Process%20-%20Outputs%20-%20Process.png?raw=1)

Talk about a tossing a pebble into a lake. Cumulative effects of updating a Project charter could have an effect on (and my count could be wrong) 40 / 47 existing processes.

I could keep diving in and performing more analysis on this but I think it's pretty suffice to say that we've got some insane impacts that I would not have even thought of by simply reading the PMBoK.  

My appreciation for graph databases is growing quickly. Hopefully I'll be able to spend some time in future projects using them so I can learn more and hopefully creating something neat.

**Edits**

2017-05-26 : Fixed typo : Project management plan -> Project charter


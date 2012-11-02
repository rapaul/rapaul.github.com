---
comments: true
date: 2009-09-13
layout: post
slug: a-behavioural-world
title: A Behavioural World
tag:
- bdd
- features
- javascript
- scenarios
- twitter
---

Or at least a behavioural twitter world,

{% img /attachments/abehaviouralworldintro.png %}

[http://github.com/rapaul/abehaviouralworld](http://github.com/rapaul/abehaviouralworld) is a little project just launched which attempts to distill behaviours from twitter. Given my recent rapture in the world of [BDD](http://en.wikipedia.org/wiki/Behaviour_driven_development), the behaviours we are distilling are those that match the scenario format.

```
Given ...
When ...
Then ...
```    

If you look carefully at the page source, you will notice a large portion of it is dedicated to an HTML comment containing the list of features and the breakdown into scenarios.  To keep things easily accessible, each time I thought of a scenario I added it to the page, once completed, it was marked with [Done].

For example, starting with the most important feature:
```
Feature 1)
In order to see how BDD syntax is being used on twitter
As a BDD zealot
I want to see tweets that contain 'given', 'when' & 'then', in that order.
```  
I then broke this feature down into scenarios:
```
[Done]
Given the search returns more than matching 10 tweets
When the user views the page
Then the 10 most recent tweets should be displayed
...
```

Each time a scenario was satisfied, I marked it with [Done]. Not all scenarios ended up being all that important, the following scenario is still pending.
```
Given the search returns no matching tweets
When the user views the page
Then instead of showing any tweets a message should be shown
```

In fact, if we attempt to tie this scenario back into the vision of the website (distilling behaviours in twitter), then we realise it adds no value. If no one is tweeting about behaviours, then it is highly unlike that anyone will be visiting our behaviour distillery.

There are a couple of features (3 & 4) in the source which haven't yet been implemented.
```
Feature 3)
In order to emphasis the given/when/then syntax
As a BDD zealot
I want to see give/when/then highlighted in the displayed tweets

Feature 4)
In order to encourage BDD syntax usage on twitter
As a BDD zealot
I want to be able to tweet from this page using a BDD template
``` 

If anyone would like to work on these features, or even just break down some useful scenarios, please feel free to fork the [code on github](http://github.com/rapaul/abehaviouralworld), provide me with a diff or add a comment (note at the time of writing github looks to be having some caching issues on the 'Source' tab).

**What no executable scenarios?**
Oops, nope. I'm yet to get into the world of writing executable scenarios in Javascript. If someone has some knowledge in this area, please feel free to fork the [github repository](http://github.com/rapaul/abehaviouralworld) and start automating the scenarios from the page source.

PS: This idea was partially inspired by a [tweet by soulnafein](http://twitter.com/soulnafein/status/3712851157) which read: 
_My girlfriend is taking the mickey out of me because recently I started explaining things using the Scenario format (Given When Then) :-)_

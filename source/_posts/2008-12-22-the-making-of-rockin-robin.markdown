---
comments: true
date: 2008-12-22
layout: post
slug: the-making-of-rockin-robin
title: The making of Rockin' Robin
tag:
- dojo
- dojotoolkit
- javascript
- json
- jsonp
- last.fm
- mashup
- rockin' robin
- social
- twitter
- web
---

This post details the making of Rockin' Robin, a client side mashup between last.fm and twitter.  If you missed my introduction to Rockin' Robin, you can read the post [here](http://www.rapaul.com/2008/11/11/rockin-robin/). Or you can try it out @ [http://robin.rapaul.com/](http://robin.rapaul.com/)

**Querying**

What makes this mashup special is that the entire mashup runs client side, all requests to the last.fm and twitter services are made in the user's browser.  Being a purely in-browser mashup there are some restrictions as to what requests can be made, specifically current browsers do not support cross-site requests for regular XmlHttpRequests (XHR).  While this is a proposed feature for upcoming browsers, we need a solution that works with current browsers.

Enter [dojo.io.script.get()](http://api.dojotoolkit.org/jsdoc/dojo/1.2/dojo.io.script.get), Dojo's workaround for cross-domain requests.  The general gist of this hack is that it adds a script block to the page with the src attribute set to the query URL, the server then returns the JSON data as the content of the script block, this technique is known as [JSONP](http://en.wikipedia.org/wiki/JSONP#JSONP).  Fortunately Dojo hides the complexity of this hack behind a similar interface to the regular request ([dojo.xhrGet()](http://api.dojotoolkit.org/jsdoc/dojo/1.2/dojo.xhrGet)).

**Last.fm**

Accessing last.fm's API requires a registration key, which can bet set up on [http://www.last.fm/api](http://www.last.fm/api) This page also details all the available API methods broken down into areas.  For the Rockin' Robin mashup we are concerned with finding the most recently scrobbled tracks which are accessible via the [user.getRecentTracks](http://www.last.fm/api/show?service=278) method.  Below is a code snippet for making the request to load the data from the last.fm API.

[gist id="36396"]

The data returned by this request includes that meta information for the last 5 tracks submitted to the user's last.fm profile.  This information includes the artist, album (including cover art links), track name as well as when the track was played.  This information is used to build up the recent track HTML as shown below.


![lastfm-data-visualised](http://www.rapaul.com/wp-content/uploads/2008/12/lastfm-data-visualised.png)


**Twitter**

The next step is to query twitter for tweets that contain the keyword artist and title combination.  Twitter's API can be viewed at [http://apiwiki.twitter.com/Search+API+Documentation](http://apiwiki.twitter.com/Search+API+Documentation).  No registration key is required, simply make a call to their public search API, e.g.  [http://search.twitter.com/search.json?q=MGMT+Kids](http://search.twitter.com/search.json?q=MGMT+Kids)

[gist id="38897"]

It is now simply a matter of extracting the required data to build the elements to add to the page.

**Building the HTML**

Dojo provides a handy method for setting attributes on nodes ([attr](http://api.dojotoolkit.org/jsdoc/dojo/1.2/dojo.NodeList.attr)), the API was designed for existing nodes, however it can be used by passing in a newly created node. Here we use dojo.query to return a nodelist which gives access to the attr function then returns the first item (the node we just created).

[gist id="38917"]

I've heard from [@amatix](http://twitter.com/amatix) that a [Builder](http://github.com/madrobby/scriptaculous/wikis/builder) equivalent is slated for a post 1.2 release of Dojo which will hopefully do away with the above build function and make element creation tasks simpler.

**Bookmarking Support**

To allow users to bookmark and link to their Rockin' Robin page, the username is stored in the URL hash. This is a two stage process, the first is to append the username to the URL, e.g. [http://robin.rapaul.com/#GexNZ](http://robin.rapaul.com/#GexNZ) The below snippet strips off any existing hash and adds '#username' to URL.

[gist id="38909"]

The username also needs to be extacted from the URL when the page is first loaded.  The username is then used to populate the username text box and kick off the mashup.

[gist id="38912"]

That wraps up the major technical details of Rockin' Robin, you can browse the full source on [github](http://github.com/rapaul/rockinrobin/tree/master).

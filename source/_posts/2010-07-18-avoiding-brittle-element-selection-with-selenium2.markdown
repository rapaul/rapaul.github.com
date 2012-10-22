---
comments: true
date: 2010-07-18 22:31:20
layout: post
slug: avoiding-brittle-element-selection-with-selenium2
title: Avoiding Brittle Element Selection with Selenium2
wordpress_id: 608
tag:
- css
- selenium
- testing
---

By automating acceptance tests we are striving to cut back on the maintenance effort of our software. The automated tests perform a majority of the checks thus freeing up time for exploratory testing.  However care needs to be taken when automating acceptance tests to ensure they remain flexible in the face of change.  Brittle tests, which fail for seemingly unrelated reasons, can place a huge maintenance burden on a testing team negating, some of the gains automation brings.

But fear not, there are steps you can take to ensure your tests are flexible to change.  The rest of this post will give examples using [Selenium2](http://code.google.com/p/selenium/), however most of these suggestions can be applied to any browser automation framework.



### Naming Elements



Selenium2 provides an API for selecting elements on the page for inspection, for example you can find the first element with a given class using the following syntax:

[cc lang="java"]browser.findElementByClassName('theClassName')[/cc]

Similarly finding multiple elements with the same class is done by simply calling [cci]findElements[/cci]

These methods provide convenient access to elements on the page but we need to ensure the classes or IDs are semantic to ensure our selenium tests continue to function if the layout of the page changes.  Below is an example of how to build a navigational element and highlights semantic and brittle IDs for the elements.

[![](http://www.rapaul.com/wp-content/uploads/2010/07/navigation.png)](http://www.rapaul.com/wp-content/uploads/2010/07/navigation.png)

As you can see the navigation is broken down into a left and a right side. In order to select these sides a naïve approach would be to assign IDs of [cci]leftNav[/cci] and [cci]rightNav[/cci].  While these two navigational elements can be easily selected using [cci lang="java"]browser.findElementById('leftNav')[/cci] and [cci lang="java"]browser.findElementById('rightNav')[/cci], using these names can have a negative impact further down the line.

First off imagine a new requirement is added whereby the right navigation should now be shown below the left navigation, this is already becoming confusing.  As a result any tests we have that needed to locate the right navigation now need to be updated to locate the [cci]bottomNav[/cci].  Similarly any stylesheets will need to be updated to reflect this same change.  We can see that by choosing a name that is related to how the navigation looks rather than what it represents becomes a maintenance nightmare.

If in the first place we chose an ID that reflected the intention of the element, our tests would continue to work regardless of the layout of the page.  So how do we go about choosing an appropriate ID for the element?  By thinking of the purpose of the element we can come up with a more semantic name.  In this case all of the elements in the right navigation are related to the user, so we can simply assign an ID of [cci]userNavigation[/cci], no matter where the user navigation resides, be it on the right or in the footer our Selenium tests will continue to locate the element correctly. A huge save on the maintenance effort.

Another important factor to consider when naming an ID or class for an element is how your users and other stakeholders refer to the element.  By ensuring all stakeholders refer to the element in the same way 'the user navigation' we can reduce translation between user, developers, testers and the code base (selenium, view templates, stylesheets, domain model, etc).  This kind of controlled vocabulary is known in Domain Driven Design as a [Ubiquitous Language](http://domaindrivendesign.org/node/132).



### Selecting Options


A common task when filling out a form is to select an option from a drop down list. For example you may need to select your favourite country and the list contains UK, USA, NZ.  There are number of ways to select New Zealand.
[cc lang="html"]

  United Kingdom
  USA
  New Zealand

[/cc]

The worst possible solution is to grab all the options and select the country based on its position in the list.
[cc lang="java"]
browser.findElementByTagName("option")[2].setSelected() // select NZ
[/cc]
This check is likely to break if any number of small changes are made, for example the sorting may be changed to alphabetical where NZ would be the first option.  If Australia were to be added to the list NZ could be bumped to the 4th option.

To ensure our automation is not tied to the number of elements nor the ordering of the list we can make use of CSS's [attribute selector](http://www.w3.org/TR/CSS2/selector.html#attribute-selectors).
[cc lang="java"]
browser.findElementByCssSelector("option[value=NZ]").setSelected()
[/cc]
As you can see the code snippet shows the intent of the action much better, no comment is needed to qualify which option is actually being selected. As long as one of the options in this list has a value of "NZ".

If the labels for the options are the same as the submitted parameters the value attribute is often left off, in this cases we do not have an attribute to target with our attribute selector.  Instead we have to check the actual text of the options.
[cc lang="java"]
browser.findElementsByTagName('option').find { it.text == 'New Zealand' }.setSelected()
[/cc]
A little more code but at least we aren't selecting based on the position in the list. Note that we now need to find all elements and further refine with Groovy's [cci]find[/cci] method.


### A Note on XPath


Selenium supports XPath for selecting elements, e.g.
[cc lang="java"]
browser.findElementByXPath('/html/body/div/div[2]/div/ul')
[/cc]
The above example is the XPath Firefox will give you when you select the trending topic element on Twitter's homepage.  This is exactly the type of XPath selector you want to avoid, at no point in the selector string is there a mention of the meaning of any of the elements we are selecting.  In fact if a [cci]div[/cci] were to be added or removed anywhere between the trending topics and the root of the homepage the selection would fail.  Of course XPath can be written much better.
[cc lang="java"]
browser.findElementByXPath('//*[@class="trendscontent"]')
[/cc]
However I would argue that the CSS selector is simpler to read and is more inline with the standard for web pages.  The fact that the element may already have appropriate IDs and classes to allow CSS selectors to work for styling purposes is a bonus.
[cc lang="java"]
browser.findElementByCssSelector('.trendscontent')
// or
browser.findElementByClassName('trendscontent')
[/cc] 

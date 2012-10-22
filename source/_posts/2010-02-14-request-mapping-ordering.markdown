---
comments: true
date: 2010-02-14 17:30:16
layout: post
slug: request-mapping-ordering
title: Avoiding ambiguities when using @RequestMapping
wordpress_id: 437
tag:
- annotation
- java
- mvc
- params
- request
- requestmapping
- spring
---

I've recently been using the param attribute of the [@RequestMapping](http://static.springsource.org/spring/docs/3.0.x/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html) annotation from Spring MVC.

Take the case of a search feature for a website.  If a 'refine' parameter is present, the user should be shown a search form to refine their criteria. However if the refine parameter is not present the user is shown the search results for the query.
[cc lang="java"]
@RequestMapping(value="/search", method=RequestMethod.GET)
public class SearchController {

  // Handles search?query=dog&refine;=true
  @RequestMapping(params="refine=true")
  public void refine() { ... }

  // Handles search?query=dog
  @RequestMapping
  public void search() { ... }

}
[/cc]

While this code will initially work, there is some ambiguity in the way the URLs are matched.  The order in which the @RequestMapping mappings are defined in the class file is actually determining which method will be called.  If we were to define the search() method first, then a request to search?query=dog&refine;=true would satisfy the conditions for the search() method's @RequestMapping and thus the params="refine=true" on the refine() method has no bearing on the outcome of the request.

To safeguard the controller against the potential reordering of methods in a class (which in usual Java programming has no affect on the outcome of running code), we need to ensure the parameter mapping is explicit.

[cc lang="java"]
// Match if 'refine' parameter is present
@RequestMapping(params="refine=true")
public void refine() { ... }
// Match if 'refine' parameter is not present
@RequestMapping(params="!refine=true")
public void search() { ... }
[/cc]

By explicitly checking the refine parameter is not present for the search method's request mapping (!refine=true), the declaration order of our @RequestMapping annotations in the class file has no bearing on which method handles the request thus making our code robust to the reordering methods.  Similar precautions should be taken with the other attributes of @RequestMapping including method and headers.

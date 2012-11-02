---
comments: true
date: 2011-02-27
layout: post
slug: vertical-slicing
title: Vertical Slicing
tag:
- bdd
- cucumber
- stories
- verticalslicing
- wip
---

Vertical slicing is one of the key features I see in being able to deliver features rapidly, with the code developed being clean and simple to extend to future requirements. The basic premise is that instead of dividing work into horizontal slices where a task is something like _create a database schema_.  You instead slice your work into small user stories where each story is something that can be demonstrated to a stakeholder on its own.  These stories will often touch multiple 'layers' of the application such as the view layer, service layer and persistence layer.

The benefits of this approach are plenty,

* Each story delivers value, no matter how small.
* Work in progress (WIP) is reduced as tasks such as 'create a schema' are not left floating around until the tasks that utilise them are completed.
* You can focus your development effort on purely delivering just enough code to satisfy the story, without adding code you think you may need (YAGNI).
* Tight feedback loop.
* Reduced merge conflicts - your local code diverges from master for shorter spans.

Slicing stories vertically fits well with the Outside-In approach favoured by Behaviour Driven Development. Using an Outside-In approach you take a user story, build some acceptance criteria around it (potentially automated with a tool like [Cucumber](http://cukes.info/)), then work from the outside in to complete the story. e.g In a web application I like to  start with the view layer, building in enough of the view to satisfy the acceptance criteria.  The view layer now defines the acceptance criteria for the layer beneath it, in this case the controller.  This process continues with each layer creating a pull signal to write more code at a lower level. Development continues in a Test Driven style until the acceptance criteria for the story is met.

An example case I like to give for vertical slicing is for a search feature we implemented at f1000.com.  I talk briefly about it in my talk on [Acceptance Testing with Geb](http://www.rapaul.com/2011/01/26/agile-acceptance-testing-with-geb/) (39 minutes in). The search feature contains a number of different stories - developing the entire feature behind closed doors without a tight feedback loop would invariably result in building the wrong thing, even if it was what was originally asked for.  Instead we took an iterative approach with thin vertical slices for each story.

The first cut of the search feature was designed to offer the greatest value, allowing users to search by keyword. The user could not go to the second page of results, change the sort order or any of the other features we ended up with. By getting a first cut out to our stakeholders early on they could immediately play with search and provide feedback on the direction.  We continued in such a fashion adding support for more stories based on the importance, for example multi-domain search (evaluations, reports, faculty, blog) adds more value (and was done first) than being able to move to the next page in the results as the most relevant hits should appear on the first page.

Related reading:
[http://www.energizedwork.com/weblog/2005/05/slicing-cake.html](http://www.energizedwork.com/weblog/2005/05/slicing-cake.html)

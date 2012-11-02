---
comments: true
date: 2009-03-02
layout: post
slug: unit-testing-fundamentals
title: Unit Testing Fundamentals
tag:
- coverage
- dependency injection
- eclipse
- java
- JUnit
- mocking
- mockito
- NUnit
- rhino mocks
- seams
- tdd
- testability
- unit testing
- vb.net
---

Below and [attached](/attachments/unit_testing_fundamentals.pdf) are slides from a presentation I gave to colleagues at Kiwiplan.  The presentation covers a range of topics regarding unit testing best practices, including test driven development (TDD), mocking frameworks, avoiding over-specification and test coverage.  The slides also include diagrams to help explain the concept of isolating code through the use of seams.

<iframe src="http://www.slideshare.net/slideshow/embed_code/1076073" width="512" height="421" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC;border-width:1px 1px 0;margin-bottom:5px" allowfullscreen> </iframe>

I've attached the final outcome of the live TDD examples.  Obviously TDD doesn't come across very well as a final solution, however the code also provides basic examples of the [Mockito](http://www.mockito.org) and [Rhino Mocks](http://ayende.com/projects/rhino-mocks.aspx) libraries.  The code style is based on Spring Framework's annotation driven MVC framework.

[Java example code](/attachments/java-unit-testing-example.zip) - JUnit, Mockito, EclEMMA (Eclipse Java coverage)
[VB.Net example code](/attachments/vbnet-unit-test-example.zip) - NUnit, Rhino Mocks

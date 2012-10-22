---
comments: true
date: 2009-06-11 21:19:39
layout: post
slug: review-spring-web-flow-2
title: 'Review: Spring Web Flow 2 Web Development'
wordpress_id: 231
tag:
- java
- jsf
- review
- spring
- testing
- web
- webflow
---

[![spring-webflow-2](http://www.rapaul.com/wp-content/uploads/2009/06/spring-webflow-2.png)](http://www.rapaul.com/wp-content/uploads/2009/06/spring-webflow-2.png)
[Spring Web Flow 2 Web Development](http://www.packtpub.com/develop-powerful-web-applications-with-spring-web-flow-2/book)
Authors: Markus Stäuble & Sven Lüppken
Publisher: [Packt Publishing](http://www.packtpub.com/)
Date of Publishing: March 2009
Sample Chapter: [Chapter 4: Spring Faces](http://www.packtpub.com/files/spring-web-flow-2-web-development-sample-chapter-4-spring-faces.pdf)





## Intended Audience


This book is intended for Java web application developers who wish to learn about [Spring Web Flow 2](http://www.springsource.org/webflow).  A base knowledge of the Spring Framework and its associated MVC is advised, however most examples in the book provide sufficient detailing of the surrounding technologies and their integration points.  This allows the reader to follow the examples without looking up details from other reference sources.



## Chapter 1 - Introduction


The first chapter of this book provides a general introduction to Spring Web Flow 2 including some of the basic terminology.  For those familiar with Spring Web Flow 1, there is a useful section detailing the major changes introduced in Web Flow 2.  Having not used Spring Web Flow in over a year, I found the summary provided enough information to reacquaint myself.



## Chapter 2 - Setup for Spring Web Flow 2


Chapter 2 details how to get Spring Web Flow setup in a Spring web application.  Details on where to find the source code and examples provides a good starting point for users who wish to see Web Flow in action before delving into the details.  An explanation of tooling support, such as setting up Eclipse to visualise flow, is a helpful addition especially with the prevalent screenshots.



## Chapter 3 - The Basics of Spring Web Flow 2


As expected, this chapter covers the necessities for building a Web Flow, focussing on defining the flow using XML.  This is by far the most important and useful chapter of the book.  The chapter is just over 50 pages long, but is well worth the read as most sections are fundamental in using Web Flow, topics include: flow descriptors (including a diagram), flow scoped persistence, the newly supported EL expression language, scope levels, @Autowired behaviour gotchas, inputs & outputs, subflows and end states.



## Chapter 4 - Spring Faces


Not being a JavaServer Faces (JSF) user, I skimmed over this chapter, however it is available as a [sample chapter](http://www.packtpub.com/files/spring-web-flow-2-web-development-sample-chapter-4-spring-faces.pdf).



## Chapter 5 - Mastering Spring Web Flow


This chapter contains a mix of topics.  The first section includes further information on the usage of subflows, however I feel the explanation is a little brief.

The next section talks about integration with the Spring Javascript project.  Spring Javascript is explained as being an abstraction layer on top of existing toolkits (only Dojo supported so far), but the example given has a hardwired dependency on dijit.form.DateTextBox.  I'm not sure if this is just a bad example or if that is how Spring Javascript works.  An introduction to the Tiles framework is then given and tied in nicely with an explanation of how to load page fragments using Spring Javascript.

The final section in this chapter covers advanced configuration of flows. I would advise using this section purely as a reference as it is heavy on detail.



## Chapter 6 - Testing Spring Web Flow Applications


Being a bit of a testing zealot, this was the section I was really hanging out for.  The opening sentence reads, "Testing is an important aspect in every software development process".  Off to a good start, unfortunately this chapter didn't live up to my expectations.

The Web Flow specific test setup and assertions are explained well, for example calls such as context.setEventId("login"), this.resumeFlow(context) and assertFlowExecutionEnded().

The example then introduces [EasyMock](http://easymock.org/) to help isolate and focus the unit tests.  However this example has a glaring issue; it creates a mock of the wrong object.  The example models a user login using a UserService to fetch a user by username.  You would expect the mocked object in this case to be the UserService, but the example mocks out the underlying EntityManager.  As a result the unit test for the flow ends up setting expectations on Hibernate HQL, thus polluting the flow's unit test with database queries that should really be tested at the service or DAO layer.  If you change the way users were fetched in the DAO (or you moved away from Hibernate) you wouldn't expect your Web Flow tests to be affected.

My advice when reading this chapter is to skip to the section titled _More testing with EasyMock_.  This section shows correct usage of EasyMock and demonstrates a test driven approach to testing.



## Chapter 7 - Security


The final chapter covers the integration of Spring Security, the only real option for security in Spring.  The details in this section are largely not specific to Spring Webflow, however they are important for developing any non-trivial application and are well explained with examples.



## Conclusion


I would recommend this book to any developer new to Spring Web Flow.  The concepts and much of the implementation between Web Flow 1 and 2 remains the same, meaning this isn't an essential read for existing Web Flow users.

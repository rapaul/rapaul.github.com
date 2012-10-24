---
comments: true
date: 2010-05-03
layout: post
slug: cucumber-world-with-groovy
title: Cucumber World with Groovy
tag:
- bdd
- cucumber
- cuke4duke
- groovy
- mixin
- world
---

[Cucumber](http://cukes.info/) provides the ability to create a _world_ in which you can expose useful methods for the testing environment.  This post briefly details how to expose these methods when using Groovy with [Cuke4Duke](http://wiki.github.com/aslakhellesoy/cuke4duke/).

```
Feature: World
In order to make useful methods available
As a tester
I want the world to be full of wonderful methods

Scenario: Greet
When I greet
...

Scenario: Farewell
When I farewell
...
```

These scenarios capture the ability to greet and farewell our customers.  The matching step definitions are shown below.

``` groovy
// hello_steps.groovy
import static org.junit.Assert.*
import static org.hamcrest.CoreMatchers.*

this.metaClass.mixin(cuke4duke.GroovyDsl)

When(~'I greet') {
  assertThat(hello(), is('hi'))
}

When(~'I farewell') {
  assertThat(goodbye(), is('ciao'))
}
```

You will notice that we haven't defined the _hello()_ or _goodbye()_ methods anywhere in our step definition file.  We are assuming these methods are provided by the _world_ environment that Cucumber provides.  This allows the _world_ methods to be shared between many different step definition files.

Next we define the _world_ environment.  Instead of defining all methods in a single class, we can group the methods into individual classes and make them all available on the _world_ class at runtime using [Groovy's mixin](http://groovy.codehaus.org/Runtime+mixins) support.

``` groovy
// env.groovy
this.metaClass.mixin(cuke4duke.GroovyDsl)

class Greeter {
  def hello() {
    'hi'
  }
}

class Fareweller {
  def goodbye() {
    'ciao'
  }
}

class World {}

World() {
  World.mixin Greeter, Fareweller
  new World()
}
```

The _World()_ call is part of Cucumber's DSL, here we use it to publish an instance of our _World_ class thus making both _hello()_ and _goodbye()_ available in the testing environment (note that the name of the _World_ class is not important).

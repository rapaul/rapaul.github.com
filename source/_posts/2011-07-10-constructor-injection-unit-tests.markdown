---
comments: true
date: 2011-07-10
layout: post
slug: constructor-injection-unit-tests
title: Constructor injection, and how it simplifies unit test setup
tag:
- collaborators
- constructor
- goos
- java
- JUnit
- mockito
- oop
- unit testing
---

I've recently been reading [Growing Object-Oriented Software Guided by Tests](http://www.growing-object-oriented-software.com/) (GOOS), and one (of the many) aha moments was a piece of test code that mocked the collaborators and instantiated the object under test - all in the declaration of the test's private fields.  I am particularly fond of this approach for two reasons:

 * The test code setup is minimal and easily scanned
 * This approach encourages all required collaborators to be passed in through the constructor (aka constructor injection)

I've included an illustrative example below using Mockito, the actual test isn't important but it proves this setup style works.

``` java
import static org.mockito.BDDMockito.*;
import org.junit.Test;

public class ItemCheckerTest {
	
	private final ItemFetcher itemFetcher = mock(ItemFetcher.class);
	private final Notifier notifier = mock(Notifier.class);
	private final ItemChecker itemChecker = new ItemChecker(itemFetcher, notifier);
	
	@Test
	public void notifiesStoreManager() throws Exception {
		given(itemFetcher.fetch()).willReturn(new FetchedItem());
		
		itemChecker.check();
		
		verify(notifier).notifyStoreManager();
	}

	...
}
```

For those unfamiliar with Mockito, the `given` call stubs a query, while the `verify` call uses a [test spy](http://xunitpatterns.com/Test%20Spy.html) to check a command call was made.

The important lines are the 3 private member variables of the test class, the first 2 use Mockito's mock method to instantiate test doubles for our collaborators. The 3rd member variable (itemChecker) is the object under test, you will notice that it is instantiated with both of its required collaborators in the constructor.  These 3 lines perform all the wiring we require for our test, without having to resort to `@Before` methods to set properties.

The reason we can leverage the member variables for this setup is that [JUnit creates a new instance](http://martinfowler.com/bliki/JunitNewInstance.html) of ItemCheckerTest for each of the test methods (`@Test`). Providing each test with its own set of collaborators ensuring each test runs in isolation.

The most important side effect of setting up the test code in this fashion is that it promotes the use of constructors for wiring up collaborators. Using the constructor for collaborators has a couple of very appealing aspects:

 * It becomes impossible to create circular dependencies between your objects
 * Your objects are less prone to wiring bugs as they are upfront about their required collaborators.

Why would you want to be upfront about your collaborators, Steve Freeman & Nat Price (GOOS) have this to say:

> 
Partially creating an object and then finishing it off by setting properties is brittle because the programmer has to remember to set all the dependencies. When the object changes to add new dependencies, the existing client code will still compile even though it no longer constructs a valid instance. At best this will cause a NullPointerException, at worst it will fail misleadingly.



Miško Hevery also has a great blog post on [constructor vs setter injection](http://misko.hevery.com/2009/02/19/constructor-injection-vs-setter-injection/).

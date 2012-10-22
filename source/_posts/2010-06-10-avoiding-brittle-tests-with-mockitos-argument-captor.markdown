---
comments: true
date: 2010-06-10
layout: post
slug: avoiding-brittle-tests-with-mockitos-argument-captor
title: Avoiding Brittle Tests with Mockito's ArgumentCaptor
tag:
- argumentcaptor
- groovy
- java
- mocking
- mockito
---

One of the core principles behind my love of [Mockito](http://mockito.org) is its ability to avoid brittle tests, by brittle tests I mean unit tests which fail when seemingly unrelated functionality changes.

Below I will outline one of Mockito's lesser known features, the [ArgumentCaptor](http://mockito.googlecode.com/svn/branches/1.8.3/javadoc/org/mockito/ArgumentCaptor.html) that shines in certain use cases.

A common requirement for a web application is to send emails to users. In this case our web application is a travel booking system.  If you make use of a templating language such as Velocity your email generation service might look similar to the following:

[cc lang="java"]
interface Mailer {
  void send(
    String to,
    String templateName,
    Map<String, Object> model)
}
[/cc]

Note: All examples are shown in Groovy for brevity, but the examples apply perfectly well to Java.

Imagine the email we send is simply welcoming the user to our web site upon registration

`
Welcome $name,

Thanks for signing up!
Book your holiday at http://example.com/
`

As you can see the only dynamic element required in the model map is the user's name.  A unit test to define the behaviour of our registration service will simply verify that a map is passed with the user's first name.

[cc lang="java" lines="150"]
private Mailer mailer
private RegistrationService registrationService

@Before
void setUp() {
  // Inject the mocked mailer
  mailer = mock(Mailer)
  registrationService = new RegistrationService(mailer)
}

@Test
void shouldSendRegistrationEmailWelcomingUser() {
  // given a new user
  User user = new User("Jim", "jim@example.com")

  // when the user registers
  registrationService.register(user)

  // then an email should be sent welcoming the user
  verify(mailer).send("jim@example.com", "welcome.vm", ["name":"Jim"])
}
[/cc]

As you can see the test starts off simple, we are verifying the mailer's send call was invoked with the correct email address, template name and model data.  In this case the model simply contains the name of the newly registered user.

A new requirement arrives which states we want to include the latest holiday deals in the welcome email.

`
Welcome $name,

Thanks for signing up!
Book your holiday at http://example.com/

Check out our latest travel offerings:
#foreach($offer in $offers)
  
#end
`

In order to test this requirement, a naive approach would be to simply add the travel offering assertions into the first test we created.  This has the downside that the test is no longer specific to a particular requirement, as we add more content to our email the test would continue to grow and become unwieldy (especially if any conditional logic exists).  We want to keep our tests focussed by limiting each test to a [single logical assert](http://laribee.com/one-logical-assertion-per-test).

Below we create a second test specific to the inclusion of the latest offers.

[cc lang="java"]
@Test
void shouldSendRegistrationEmailWithLatestTravelOfferings() {
  // given a new user
  User user = new User("Jim", "jim@example.com")
  // and the latest travel offers
  def offers = ["Offer 1", "Offer 2"]
  given(latestOffers.get()).willReturn(offers)

  // when the user registers
  registrationService.register(user)

  // then an email should be sent with the latest travel offers
  verify(mailer).send("jim@example.com", "welcome.vm", ["offers":offers])
}
[/cc]

At first glance this looks like a good test, we are checking the latest offers are included in the model so they can be rendered in the email template.

Unfortunately both tests will fail.

In order to verify the correct calls are made, Mockito uses the equality (equals) method of the passed arguments.  In the above case we are checking the equality of two strings and a map.  It is of course the equality of the model map that is causing the test to fail.

shouldSendRegistrationEmailWelcomingUser() fails as Mockito is expecting a map containing simply the user's name [name:"Jim"], but due to the added travel offerings the map is actually [name:"Jim", offers:offers].  The same failure applies to shouldSendRegistrationEmailWithLatestTravelOfferings() as it is verifying the map only contains offers.

As mailer.send() has no return type (void) we have no simple way to access the model map in our test.  However Mockito offers a couple of ways around this, the first is the creation of a custom Matcher.  The second is to use an ArgumentCaptor which is the approach I will be using today.

[![Gotta catch 'em all!](http://www.rapaul.com/wp-content/uploads/2010/06/pokeball1.png)](http://www.rapaul.com/wp-content/uploads/2010/06/pokeball1.png)

The ArgumentCaptor is a specialised ArgumentMatcher that records the matched argument for later inspection. 

[cc lang="java" lines="150"]
@Test
void shouldSendRegistrationEmailWelcomingUser() {
  // given a new user
  User user = new User("Jim", "jim@example.com")

  // when the user registers
  registrationService.register(user)

  // then an email should be sent welcoming the user
  ArgumentCaptor model = ArgumentCaptor.forClass(Map)
  verify(mailer).send(eq("jim@example.com"), eq("welcome.vm"), model.capture())
  assertThat(model.value.name, is("Jim"))
}
[/cc]

Firstly an ArgumentCaptor is created for the class, in this case Map, we wish to inspect.  The ArgumentCaptor is then used as an ArgumentMatcher in the verify call.  No matter what keys or values the model map contains, the ArgumentCaptor will always match thus allowing the verify call to succeed.  Now that we have captured the model we can inspect it by calling getValue() (simply value in Groovy) to access the original map that was passed to the Mailer service by our production code.  By verifying only the key/value pairs that are specific to our test (the user's name) we can ensure that any other pieces of information that are added to the map in the future don't affect any of the existing tests.

The astute reader may have noticed the inclusion of equality matchers for both the email address eq("jim@example.com") and the template name.  While Mockito relies on the equality method of the arguments by default, if any of the arguments are an ArgumentMatcher, then all of the arguments must be a matcher.

In summary, the ArgumentCaptor allows tests to remain focussed with a single logical assert, even when the object you wish to inspect is created in the code under test and passed to a collaborator via a void method call.

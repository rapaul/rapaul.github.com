---
comments: true
date: 2010-03-16 21:13:29
layout: post
slug: cucumber-session-fixation
title: A Cucumber Scenario for Session Fixation Attacks
wordpress_id: 470
tag:
- bdd
- cucumber
- cuke4duke
- scenario
- security
- session fixation
- spring
- webdriver
---

When working on a Spring project I was alerted via the logs that my login handling was susceptible to [Session Fixation Attacks](http://en.wikipedia.org/wiki/Session_fixation).  It was nice of Spring to alert me of this, and it seems like the [problem](http://markmail.org/thread/o7f6d5gkle4kqqbt) is due to the Apache httpd connection via mod_jk.  But before diving into a solution I decided to write a simple acceptance test using [Cucumber](http://cukes.info/).  This allows me to first prove the security hole is present, easily tell when I have fixed it and has the additional benefit of being able to add the check to a set of automated regression tests to ensure it never happens again.

The countermeasure [built into Spring](http://static.springsource.org/spring-security/site/docs/3.0.x/apidocs/org/springframework/security/web/authentication/session/SessionFixationProtectionStrategy.html) uses the [Identity Confirmation](http://en.wikipedia.org/wiki/Session_fixation#Best_solution:_Identity_Confirmation) technique.  This basically involves invalidating a user's session when they login and creating a new session with a different _JSESSIONID_.  It turns out to be fairly easy to write a test for this in Cucumber.  Firstly I wrote the plain text scenario:


    
    
    Scenario: Avoiding Session Fixation Attacks through Identity Confirmation
    Given I have a session ID
    When I sign in
    Then I should have a different session ID
    



The next step is to write the step definitions.  [Cuke4Duke](http://wiki.github.com/aslakhellesoy/cuke4duke/) provides Cucumber support for JVM languages and makes it easy to write reusable step definitions.  As I'm a fan of the expressiveness of Groovy I chose to write the step definitions using the [GroovyDsl](http://wiki.github.com/aslakhellesoy/cuke4duke/groovy).

[cc lang="java" lines="150"]
import org.openqa.selenium.By
import static org.junit.Assert.*
import static org.junit.matchers.JUnitMatchers.*
import static org.hamcrest.CoreMatchers.*

this.metaClass.mixin(cuke4duke.GroovyDsl)

def sessionId

Given(~'I have a session ID') {
  browser.get("$host/signin")
  sessionId = getSessionId()
}

When(~'I sign in') {
  def signinForm = browser.findElement(By.name('signin'))
  signinForm.findElement(By.name('j_username')).sendKeys('testing@example.com')
  signinForm.findElement(By.name('j_password')).sendKeys('123456')
  signinForm.submit()
}

Then(~'I should have a different session ID') {
  assertThat(getSessionId(), is(not(sessionId)));
}

def getSessionId() {
  browser.manage().cookies.find() { it.name == 'JSESSIONID' }.value
}
[/cc]

The _browser_ variable is injected via _env.groovy_ as an instance of [WebDriver](http://webdriver.googlecode.com/svn/javadoc/org/openqa/selenium/WebDriver.html) which provides the methods to find elements and manage cookies.

_def sessionId_ simply declares a variable allowing sharing of state between the steps so we can check the session ID has been changed.

---
comments: true
date: 2009-08-09
layout: post
slug: bddmockito-eclipse
title: BDDMockito & Eclipse
tag:
- Add new tag
- bdd
- eclipse
- java
- mocking
- mockito
- testing
- unit testing
---

On the 23rd of July, [Mockito](http://mockito.org/) 1.8.0 was released, you can see a full listing of changes in their [release notes](http://code.google.com/p/mockito/wiki/ReleaseNotes).  One feature that took my fancy was the inclusion of [BDD](http://en.wikipedia.org/wiki/Behavior_Driven_Development) aliases for stubbing an API, instead of using when, the Mockito team now encourage the use of given to bring your tests inline with the BDD style. To illustrate this, here is an example taken from the [BDDMockito javadoc](http://mockito.googlecode.com/svn/branches/1.8.0/javadoc/org/mockito/BDDMockito.html).

[cc lang="java" lines="50"]
import static org.mockito.BDDMockito.*;
 
Seller seller = mock(Seller.class);
Shop shop = new Shop(seller);
 
public void shouldBuyBread() throws Exception {
  //given  
  given(seller.askForBread()).willReturn(new Bread());
 
  //when
  Goods goods = shop.buyBread();
   
  //then
  assertThat(goods, containBread());
}  
[/cc]

By breaking the test into given, when & then, future maintainers of the tests will more quickly understand which parts of the test relate to setup, exercising the SUT and asserting.

_(stealing from Apple's [AppStore ad](http://www.youtube.com/watch?v=szrsfeyLzyg))_ What's great about Eclipse, is that if you want to write a foreach loop, there's a template for that.  If you want to create a unit test, there's a template for that. Yip there's an <del>app</del> template for just about anything.

To minimise keystrokes and encourage the use of this style, you can modify the existing Test template as shown below to automatically generate the BDD style comments & statically import the BDDMockito members.  To modify the template, navigate to Window > Preferences > Java > Editor > Templates.  Click on Test (the JUnit 4 test template) and click edit.  Paste in the follow template:

[cc lang="java" lines="50"]
@${testType:newType(org.junit.Test)}

public void ${testname}() throws Exception {
  // given ${cursor}
  ${staticImport:importStatic('org.junit.Assert.*', 'org.mockito.BDDMockito.*')}

  // when

  // then

}
[/cc]

You are now ready to start writing BDDMockito style unit tests using Eclipse's template, simply type Test, hit Ctrl+Space and you will be given the following option.

![ctrl-space](http://www.rapaul.com/wp-content/uploads/2009/08/ctrl-space.png)

Selecting the option will generate the test method, focus first given to the test name, then to the blank line after // given.

![created-template](http://www.rapaul.com/wp-content/uploads/2009/08/created-template.jpg)

There you have it, a simple Eclipse template to generate your BDDMockito style tests.

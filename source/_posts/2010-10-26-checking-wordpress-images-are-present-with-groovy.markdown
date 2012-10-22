---
comments: true
date: 2010-10-26
layout: post
slug: checking-wordpress-images-are-present-with-groovy
title: Checking Wordpress images are present with Groovy
tag:
- groovy
- wordpress
---

Below is a little script I knocked up today to ensure all uploaded images referenced in a Wordpress XML export are present on your server.  A useful check when moving to a new Wordpress instance.

[cc lang="java"]
def imagePattern = ~'http://www.rapaul.com/wp-content/uploads/.+?/.+?/.+?(jpg|jpeg|png|gif)'
new File('./wordpress-export.xml').text.findAll(imagePattern).each { address ->
   try {
       new URL(address).openStream().close()
   } catch (FileNotFoundException e) {
       println "Could not find $address"
   }
}
[/cc]

I'm sure a bash guru could hack together something similar but Groovy really does make it simple.

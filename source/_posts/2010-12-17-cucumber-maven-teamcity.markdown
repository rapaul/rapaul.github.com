---
comments: true
date: 2010-12-17 11:07:58
layout: post
slug: cucumber-maven-teamcity
title: Cucumber, Maven & TeamCity
wordpress_id: 674
tag:
- bdd
- ci
- cucumber
- cuke4duke
- groovy
- teamcity
---

Having a suite of acceptances tests is all well and good, but if they don't run regularly they tend to rot.  Much like unit tests they should be part of your continuous integration configuration.  We use TeamCity, so it was a logical choice for running our Cucumber scenarios.

**Running Cucumber with Maven**
As we predominantly use Java/Groovy all our Cucumber tests are written in Groovy and run automatically through Maven as part of the integration test phase.  Checkout the [Cuke4Duke](https://github.com/aslakhellesoy/cuke4duke/wiki) project if you want to see more details about using Cucumber on the JVM.  In particular there is a page dedicated to running [Cuke4Duke with Maven](https://github.com/aslakhellesoy/cuke4duke/wiki/Maven).  Once your pom has been configured its simply a matter of calling the integration phase:
[cc]
mvn integration-test                              # Run all features/scenarios
mvn integration-test -DcukeArgs="--tags @search"  # Run only scenarios tagged with @search
[/cc]
Locally I have a bash script that lets me simply type cuke @search to achieve the same result.
[cc]
#!/bin/bash
mvn integration-test -DcukeArgs="--tags $1"
[/cc]

**Adding to TeamCity**
TeamCity has built in support for running Maven goals, as such it's relatively trivial to get the Cucumber scenarios running.



	
  1. Create a new build configuration, on the build step select Maven2 as the runner type.

	
  2. Set the goal to integration-test

	
  3. Skip most of the remaining fields.

	
  4. In the JVM command line parameters enter:  -DcukeArgs="--strict -DcukeMaxHeapSize="-Xmx4000m" -DcukeMaxPermSize="-XX:MaxPermSize=2000m"

This should be all that is required to get the build running.  --strict tells Cucumber not to guess when a step doesn't exactly match a step definition (optional but recommended).  The max heap size and max perm size are properties configured in the pom to boost the memory allocations.  These are required by our build, but I have a sneaking suspicion it's to do with the parsing of the step definition closures (Groovy specific).

Currently we have a dedicated desktop box (Win7) registered as a TeamCity agent, this allows Cucumber to run the scenarios through the browser (via Selenium/WebDriver).

**Adding Reporting to TeamCity**
As the build configuration stands, Cucumber will output using its normal coloured terminal format.  So while you will be getting feedback on whether the build was successful from TeamCity, drilling in to see any failures will involve reading the raw Cucumber output.  Fortunately Cucumber allows us to specify the output format we require, including JUnit XML reports.



	
  1. Update the JVM command line parameters to include --format junit --out target/junit.xml

	
  2. Add an Ant JUnit report type with the reports directory set to %system.teamcity.build.checkoutDir%/target/**/*.xml


[![](http://www.rapaul.com/wp-content/uploads/2010/12/test-passed.png)](http://www.rapaul.com/wp-content/uploads/2010/12/test-passed.png)

You now get individual scenarios reported as tests in TeamCity, you can then monitor for long running tests, check stacktraces and get notified immediately via a TeamCity notifier as soon as a single scenario fails (no need to wait until the entire suite finishes).

[![](http://www.rapaul.com/wp-content/uploads/2010/12/stack.png)](http://www.rapaul.com/wp-content/uploads/2010/12/stack.png)

**Triggering Cucumber**
Each time we deploy a new version of our application to the testing/dev box we want to automatically run the Cucumber tests.  Deployment is handled via TeamCity so we have a single click to push the latest code onto the server.  Once the application has been deployed the Cucumber build is automatically trigger, this can be set on the Build Triggering step of the configuration.  Should the testing deployment not be manually trigger, the nightly build will push the latest code and run the Cucumber tests.

**Key Features**
[![](http://www.rapaul.com/wp-content/uploads/2010/12/in-the-green.png)](http://www.rapaul.com/wp-content/uploads/2010/12/in-the-green.png)

The above screenshot shows the various builds we have related to our product.  The Continuous Integration is our unit tests, these run quickly after every commit.  Next we break the Cucumber build down into 3 discrete builds. Key Features is a tag (@keyfeature) we use against features and scenarios the business identifies as being the most important, the idea being that these are run first to give us quick feedback, in this case there are 75 key features which run in just under 10 minutes.  The remaining scenarios are run in the non-key features build which runs automatically immediately after the key features have completed.  These currently take just under 50 minutes.  So the total feedback loop is roughly 1 hour.

**Work in Progress (WIP)**
The last Cucumber build is the WIP (work in progress) build, these are features and stories that are currently being worked on.  TeamCity is configured slightly differently in this case, any scenarios that pass will fail the build.  The idea being that either the scenario was written wrongly (it should fail first) or the scenario is now complete and the @wip tag should be removed.  To make Cucumber fail when any scenarios pass we we need to pass a special flag --wip in the JVM parameters.
[cc]
-DcukeArgs="--format junit --out target/junit.xml --tags @wip --wip"
[/cc]
The current value of 17 work in progress scenarios seems a bit high, ideally we'd like to eliminate waste by keeping our WIP down.  Cucumber supports such limits as a [command line argument](https://rspec.lighthouseapp.com/projects/16211/tickets/353-limiting-number-of-feature-elements-in-tagged-state).

**Experience Report**
If you'd like to hear more about Cucumber, Cuke4Duke, Groovy, Selenium & Geb I'm giving an experience report at [SkillsMatter on the 26th of Jan](http://skillsmatter.com/event/agile-testing/acceptance-testing-with-geb).

**Related reading**
[http://gojko.net/2010/01/01/bdd-in-net-with-cucumber-cuke4nuke-and-teamcity/](http://gojko.net/2010/01/01/bdd-in-net-with-cucumber-cuke4nuke-and-teamcity/)

---
comments: true
date: 2011-06-05
layout: post
slug: zero-to-headless-browser-tests-jenkins
title: From zero to headless browser tests in Jenkins
tag:
- capybara
- ci
- cucumber
- jekins
- rvm
- selenium
- virtualbox
- vnc
- webdriver
---

After spending a large portion of the day I can proudly say I have a working set of browser based tests that run on a headless Jenkins install. By headless I mean a server without any physical display installed, as is typical for server machines. This facilitates the execution of high level acceptance tests in much the same fashion as lower level unit and integration based tests, albeit at a slower rate.

{% img right /images/jenkins-headless.png 300 230 %}

The problem I am trying to solve here is a quick feedback loop on acceptance test level behaviour. This blog post will be talking about getting Cucumber scenarios running for a single browser (Firefox). Cross browser testing is a different problem, for which think [Sauce Labs](http://saucelabs.com/) would be a better solution as they take the hassle out of provisioning and maintaining a wide range of operating system and browser combinations.

Outlined below are the steps I followed to go from installing Ubuntu server edition through to running the browser based tests (with [Cucumber](http://cukes.info/), [Capybara](https://github.com/jnicklas/capybara), Selenium-Webdriver). You may find some steps are not required on your operating system or for the project you wish to test. Admittedly I dove down a few rabbit holes, but thanks to VirtualBox's snapshot feature I could safely revert if things turned sour.

 * Installing Ubuntu
 * Installing Jenkins
 * Going Headless
 * Installing Ruby with RVM
 * Installing Firefox
 * Creating a Job
 * Bonus Points, watching the browser in realtime

## Installing Ubuntu

If you have an existing server you can skip this step. If not grab yourself a copy of [Ubuntu Server edition](http://www.ubuntu.com/download/server/download). As I wanted a simple way to play with Jenkins without provisioning hardware I used VirtualBox for virtualisation. I followed the usual VirtualBox installation, however once installed the server showed a blank screen on boot. This was fixed by following the workaround on the [Ubuntu forums](http://ubuntuforums.org/showthread.php?t=1743535).

When installing, it is handy to enable the OpenSSH server so you can SSH onto the box from your desktop terminal, this makes copy & pasting some of the later steps much easier. To make the VirutalBox server visible on the network, change the network mode from NAT to bridged.

## Installing Jenkins

Installing Jenkins is a breeze, as debian packages have been set up, check the [wiki page](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu) for details.
``` bash
$ wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
$ sudo vi /etc/apt/sources.list.d/jenkins.list
Add "deb http://pkg.jenkins-ci.org/debian binary/"
$ sudo apt-get update
$ sudo apt-get install jenkins
```

This automatically creates an account called _jenkins_. We will need to login as this user later so set a password for _jenkins_ with:
```
sudo passwd jenkins
```
You should now be able to view the Jenkins dashboard at http://your.server:8080/

### Going Headless

Now that Jenkins is installed, we want to get a headless display configured for our browser based tests. First up hit _Manage Jenkins > Manage Plugins > Available_ and install the _Hudson Xvnc_ plugin (this works with Jenkins despite its name). Schedule Jenkins to restart to pickup the plugin.  Once installed this gives us the ability to start a headless display automatically when we configure our jobs, more on that later.

With Jenkins configured we need to ensure the required software is installed on the server:
```
$ sudo apt-get install vnc4server
```

vncserver requires a password to be set before it can be used, this needs to be set before Jenkins can make use of the vncserver. For this we need to switch to the jenkins user and set a password.

```
$ sudo -Hiu jenkins
$ vncserver
Enter a password, and verify it
$ vncserver -kill :1 # or whichever display the vncserver output mentioned
```

When Jenkins runs it doesn't need to know this password, but if you want to watch a running job you can connect to the running vnc session with that password and watch the tests in real time.

I initially headed down the Xvfb route but that seemed to require a lot of custom configuration in the job's build script and isn't related to the Xvnc plugin.



## Installing Ruby with RVM


The job I'm wanting to run is a set of acceptance tests written in Cucumber with automation done using Capybara (Selenium-Webdriver under the hood). So its a Ruby job, and all good Ruby jobs use [RVM](https://rvm.beginrescueend.com/). Fortunately RVM has a page on [integrating with Hudson/Jenkins](https://rvm.beginrescueend.com/integration/hudson/). I followed the recommended steps and [installed RVM](https://rvm.beginrescueend.com/rvm/install/) for a single user (jenkins).
```
$ sudo apt-get install curl bison build-essential zlib1g-dev libssl-dev libreadline5-dev libxml2-dev git-core
$ sudo -Hiu jenkins
$ bash < <(curl -s https://rvm.beginrescueend.com/install/rvm)
```
Once RVM is configured, run _rvm notes_ to find the full list of dependencies you need to install for your required version of Ruby. e.g.
```
$ source ~/.rvm/scripts/rvm
$ rvm notes
$ sudo apt-get install build-essential bison openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-0 libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev
```
Note that I didn't give the jenkins user sudo rights, so I installed all packages through my usual admin account on the server.

RVM can be configured to allow the automatic installation of Ruby versions and gemsets by adding the following to ~/.rvmrc for the jenkins user:

```
rvm_install_on_use_flag=1
rvm_project_rvmrc=1
rvm_gemset_create_on_use_flag=1
```



## Installing Firefox


Of course, a headless server isn't any good without a browser to test
```
sudo apt-get install firefox
```
This is the default browser that selenium will select.



## Creating a Job


At this point the Jenkins server should be fully configured to run headless jobs, so lets dive in and create one. Create a new _freestyle_ job. Notice there is a new option available under the 'Build Environment' section call 'Run Xvnc during build', check this to have the plugin automatically do its magic.

For my example, I didn't bother with checking projects out source control, I simply created a project in the /tmp directory. You'll want to enable the appropriate SCM plugin and configure a checkout.
Under the build section add an _Execute shell_ step with the following:
```
#!/bin/bash -e
cd /tmp/selenium-test
source "$HOME/.rvm/scripts/rvm"
[[ -s ".rvmrc" ]] && source .rvmrc
bundle install
cucumber
```
The _-e_ flag in _#!/bin/bash -e_ ensures the script stops after any errors.
You will notice that the script sources the .rvmrc file directly for the project, this ensures the correct version of Ruby is used with a gemset appropriate for your project. My .rvmrc looked something like:
```
rvm --create use ruby-1.9.2@selenium-test
```
Calling _bundle install_ automatically installs bundler, reads the Gemfile.lock and installs all required gems. Finally _cucumber_ kicks off the actual cucumber scenarios, and fingers crossed, they should pass with flying colours.

```
Feature:Headless
  In order to keep happy customers
  As a developer I want to ensure all my features continue to pass on CI

  Scenario: Headless browser
    When I check the nets without a head
    Then I the nets should be readable

1 scenario (1 passed)
2 steps (2 passed)
0m7.975s
Terminating xvnc.
$ vncserver -kill :33
Killing Xvnc4 process ID 6873
Finished: SUCCESS 
```



## Bonus Points, watching the browser in realtime


As the headless display is running in vncserver, you can connect to the vnc session and watch the tests run in real time. Just use your regular VNC client and connect to your.server:59xx where xx is the display number output on the Jenkins console for the running job. You will need to enter the password you set the first time you ran vncserver.

[Note: most/all of these instructions should work with Hudson also] 

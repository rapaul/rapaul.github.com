---
comments: true
date: 2010-11-26 10:33:58
layout: post
slug: failing-early-in-bash
title: Failing early in bash
wordpress_id: 662
tag:
- bash
- deployment
---

We use a bash script to automate deployment to our live servers, today it didn't go so well.  Drilling into the problem it appeared one of the copy commands failed due to permissions but the deployment continued anyway and we didn't notice.

Bash provides a handy feature which will stop the script if any of the commands fail (return a non-zero value).

[cc lang="bash"]set -o errexit[/cc]

Here you can see a simple example 

[cc lang="bash"]
set -o errexit  ## exit after any error (non-zero statement) 
echo before failing command
mkdir a/b
echo after failing command
[/cc]

Without this toggle enabled the script would have quite happily failed to create directory a/b then continued on with the rest of the script printing out _after failing command_.  With the toggle enabled the script stops after it fails to create the directory.

[cc]
before failing command
mkdir: cannot create directory `a/b': No such file or directory
[/cc]

Obviously in this trivial example it doesn't seem important, but when an essential part of your deployment fails you want to know about it immediately.

Further reading available at: [http://www.davidpashley.com/articles/writing-robust-shell-scripts.html](http://www.davidpashley.com/articles/writing-robust-shell-scripts.html)

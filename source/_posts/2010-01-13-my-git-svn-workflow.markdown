---
comments: true
date: 2010-01-13 22:28:03
layout: post
slug: my-git-svn-workflow
title: My git svn workflow
wordpress_id: 417
tag:
- eclipse
- git
- svn
---

Below are the steps I use when working with Git & SVN. Please note, I am in no way a Git or SVN expert, but these are the steps that seem to work for me.

**1) Checkout codebase**
This step only needs to be done once, it will pull down a local copy of the entire history allowing for very fast nagivation of revisions and allow work to be down when no connection to a central repository is available.

    
    git svn clone svn://path/to/code/ --stdlayout



**2) Create a topic branch to work on**
We now want to start work on our first story, so we create a local branch for it.  This allows us to keep all changes related to the first story out of the master branch.  By keeping the master branch clean we can create as many local branches as we wish from it allowing us to quickly fix a bug in a separate local branch to the story we are working on.  I'll come to this later.  The _-b_ option means checkout and create a branch, this is a shortcut to creating the branch then checking it out.

    
    git checkout -b story1



.. implement the first story ..

**3) Add all new/deleted/modified files to staging area**
With git you can stage just the files that are important, these staged items then make up the contents of your commit.

    
    git add path/to/resource



**4) Create a commit**
We then create a commit of the staged files in our local branch, note this differs from SVN as a commit is only local.

    
    git commit -m "My commit message"



**5) Merge changes from your topic branch into the master branch**
Ensure the master branch is up to date

    
    git checkout master
    git svn rebase



Then merge your topic branch into master

    
    git merge story1



**6) Merge conflicts**
Merge any conflicts either manually or using _mergetool_ (I'm yet to figure out how to use the _mergetool_ command)

**7) Push to SVN**
This is the bridge that takes our Git commits and pushes them to SVN.  Each Git commit will be mapped to a respective SVN commit.

    
    git svn dcommit




**Working with multiple branches**
As I mentioned earlier, one of the major advantages of Git over SVN is the ability to have multiple working branches locally, allowing you to switch between them with relative ease.  Take for example you are working on a story, it might take you a whole day to get it done.  At midday an urgent support request arrives for a bug in the live software.  Using SVN I would typically have to checkout the HEAD revision into a separate directory and import it into Eclipse as a new project.  With Git it is much simpler, I use the following steps to do the switching.

# Commit all current work to the topic branch

    
    git add path/to/noncommited/work



# Switch to the master branch and make sure it is up to date

    
    git checkout master
    git svn rebase


# Create a new branch for the bug fix

    
    git checkout -b bugfix


# Fix the bug
# Follow steps 3-7 above
# Carry on working on the story

    
    git checkout story1



**Git GUI**
You can use the [Git GUI](http://www.kernel.org/pub/software/scm/git/docs/git-gui.html) tool for most of the steps above, I find it particularly useful for steps 3 & 4.
You can also review the history and changes using gitk, this is useful for code reviews before you push your code into a master repo.

**JGit (Eclipse Plugin)**
I use the [JGit Eclipse Plugin](http://www.jgit.org/) when working in Eclipse, it provides a diff with the previous version which I find invaluable.
Install in Eclipse using the update site: http://www.jgit.org/updates

**References**
[http://blog.tsunanet.net/2007/07/learning-git-svn-in-5min.html](http://blog.tsunanet.net/2007/07/learning-git-svn-in-5min.html)
[http://stackoverflow.com/questions/190431/is-git-svn-dcommit-after-merging-in-git-dangerous](http://stackoverflow.com/questions/190431/is-git-svn-dcommit-after-merging-in-git-dangerous)
[http://www.kernel.org/pub/software/scm/git/docs/user-manual.html#resolving-a-merge](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html#resolving-a-merge)

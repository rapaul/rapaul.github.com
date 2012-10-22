---
comments: true
date: 2010-06-21
layout: post
slug: gherkin-highlighting-for-vim
title: Gherkin Highlighting for VIM
tag:
- bdd
- cucumber
- gherkin
- highlighting
- ubuntu
- vim
---

UPDATE: Check the comments, people have had more success with Brent's suggestion.

I really wanted some highlighting on my .feature files in vim, I'm no vim expert so I did a bit of googling and came up with the following:



	
  * Download cucumber.vim from http://github.com/tpope/vim-cucumber/blob/master/syntax/cucumber.vim

	
  * Save the file to ~/.vim/cucumber.vim (creating the directory if required)

	
  * vim ~/.vimrc and add the following lines
[cc]au Bufread,BufNewFile *.feature set filetype=gherkin
au! Syntax gherkin source ~/.vim/cucumber.vim
[/cc]

	
  * Make sure [cc]syntax on[/cc] is also present

	
  * Open your feature file


[![](http://www.rapaul.com/wp-content/uploads/2010/06/vim-with-gherkin1.png)](http://www.rapaul.com/wp-content/uploads/2010/06/vim-with-gherkin1.png)

If anyone knows a more _vim_ way of doing this feel free to add a comment.

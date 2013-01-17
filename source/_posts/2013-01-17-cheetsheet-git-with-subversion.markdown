---
layout: post
title: "Cheetsheet - Git with Subversion"
date: 2013-01-17 18:09
comments: true
categories: 
- git
- dvcs
- subversion
- vcs
- cheatsheet
---

Previously, I blogged about using [Git with Subversion](/blog/2012/12/13/git-workflow-for-subversion/). Sometimes though, I like concise pieces of information, without all the details. For that reason, here is a cheatsheet on the steps for using Git with Subversion:

{% codeblock lang:posh %}
git svn rebase
git branch new_awesome_feature_branch
git checkout new_awesome_feature_branch
git commit -m "this is best feature ever implemented"
git rebase master
git checkout master
git merge new_awesome_feature_branch
git svn rebase
git svn dcommit
git branch -D new_awesome_feature_branch
{% endcodeblock %}
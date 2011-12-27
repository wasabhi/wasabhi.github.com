---
layout: post
title: "Cherry picking commits with git"
date: 2011-02-09 22:17
comments: true
categories: git development
---

Today, I spent a lot of time [cherry picking](http://www.kernel.org/pub/software/scm/git/docs/git-cherry-pick.html) my way through a bunch of commits.

Before explaining further - this is the scenario we were in. 

* over about 3 weeks we've had to vendor a Ruby gem so as to modify and extend it rapidly in our Rails app 
* we've done this quickly and without tracking changes in a fork of the gem (bad!)
* we needed to contribute all our work back to the gem maintainer but we needed to carefully group related commits, and send meaningful pull requests

So, after forking the gem in question and a cheeky bit of diffing we knew the all the differences between our vendorised gem and that of the fork.

    diff -uNrw . our/app/vendor/gems/gem_name/ | less

We created a temp branch and applied all the diffs by simply copying over the entire gem recursively.

    git checkout -b temp
    cp -r our/rails_app/vendor/gems/gem_name/ .

We ran all the tests in the gem, and gladly they passed, so we removed our vendor gem directory from our application - using a symbolic link instead pointing at the gem fork.  Then we ran all our tests in our application and they passed too - which is great.

So at this point, the gem was still in the vendor/gems directory, however this pointed to a fork of the gem where many changes had been made.

The next step was to commit or interactively add hunks of code from all the changes in the temp branch.

    git commit -v file
    git add -i file

This created neatly separated commits of all changes in our temp branch.

Then we checked out the master branch and compared the temp branch to the HEAD revision.  This showed us clearly the differences between the two branches.

    git checkout master
    git diff HEAD..temp

So, we've got all our commits on a temp branch and these needed to be selectively added to new branches, allowing us to logically aggregate changes, and issue pull requests to the gem's maintainer.

So - first up create a new branch and then we started to cherry pick.

    git checkout master
    git checkout -b new_set_of_commits_for_pull_request

With a list of SHAs generated from all commits on the temp branch we could tell what we needed to cherry pick.

    git log --oneline temp

So applying these commits to a branch is easy - giving us a branch with a specified set of commits on it. 

    git cherry-pick 42f169a 9068c89 750684e d5a7ac8

Then we push this branch, and follow github's amazing pull request procedure.

    git push origin new_set_of_commits_for_pull_request

Once our changes were accepted (phew!) we created a remote pointing to the gem repo, and fetched the contents.  Then we tracked all upstream changes to this gem allowing us to merge back all upstream changes into our local master branch - using an intermediate "integrate" branch to test that all merges were fast-forwards.

    git remote add gem_name <insert repo>
    git fetch gem_name
    git branch --track gem_name gem_name/master

Git is such a powerful SCM - it'll take me a while to get my head around the possibilities.  For me, to write the above down is really important - it helps me to remember for next time!

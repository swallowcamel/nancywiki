# Git Workflow

## Overview

The general process for working with Nancy is:

1. Fork on GitHub
1. Clone your fork locally
1. Create a local branch (`git checkout -b myBranch`)
1. Work on your feature, spiking/prototyping as required (see below)
1. Rebase if required (see below)
1. Push the branch up to GitHub (`git push origin myBranch`)
1. Send a Pull Request on GitHub

You should **never** work on a clone of master, and you should **never** send a pull request from master - always from a branch. The reasons for this are detailed below.

## Before Sending a Pull Request

While working on your feature you may well create several branches, which is fine, but before you send a pull request you should ensure that you have rebased back to a single "Feature branch" - we care about your commits, and we care about your feature branch; but we don't care about how many or which branches you created while you were working on it :-)

When you're ready to go you should confirm that you are up to date and rebased with upstream/master (see "Handling Updates from Upstream/Master" below), and then:

* `git push origin myBranch`
* Send a descriptive Pull Request on GitHub
* Wait for TheCodeJunkie to merge your changes in and reformat all of your code because he has StyleCop OCD ;-)

## Spiking / Prototyping

It's quite normal, and encouraged, that during design/development of your feature you create several [spikes](http://www.extremeprogramming.org/rules/spike.html)/prototypes, which you share with the other developers (\*cough\* via the [Google Group](https://groups.google.com/forum/?pli=1#!forum/nancy-web-framework) \*cough\*) for feedback. Due to the fact that rebasing public commits is [pure evil](http://progit.org/book/ch3-6.html), and that we require you to rebase any updates from upstream/master, it is recommended that you:

* Create one or more "MyFeatureSpike" branch(es) (or words to that effect) - this makes it quite clear to other developers that this is a temporary spike branch, and if they decide to fork it for their own work they should do so in the knowledge that it will:

   a) likely be rebased, and

   b) get deleted at some point.

* When you're happy with the approach, create your real feature branch and start working on that. It is suggested that you effectively "throw away" your spike branch and start afresh with a test-first approach, but as long as you end up with good quality, well tested code this isn't enforced.

## Handling Updates from Upstream/Master

While you're working away in your branch it's quite possible that your upstream master (most likely the canonical TheCodeJunkie version) may be updated. If this happens you should:

1. Stash any un-committed changes you need to
1. `git checkout master`
1. `git pull upstream master`
1. `git checkout myBranch`
1. `git rebase master myBranch`
1. `git push origin master` - (optional) this this makes sure your remote master is up to date

This ensures that your history is "clean" i.e. you have one branch off from master followed by your changes in a straight line. Failing to do this ends up with several "messy" merges in your history, which we don't want. This is the reason why you should always work in a branch and you should never be working in, or sending pull requests from, master.

If you're working on a long running feature then you may want to do this quite often, rather than run the risk of potential merge issues further down the line.

For more information on the merits of this workflow please see RobertTheGrey's [excellent post](https://groups.google.com/forum/#!msg/fubumvc-devel/olH11f_mbk4/pGV6MqFfBSQJ) on the [Fubu MVC Development Group](https://groups.google.com/forum/#!msg/fubumvc-devel/olH11f_mbk4/pGV6MqFfBSQJ).
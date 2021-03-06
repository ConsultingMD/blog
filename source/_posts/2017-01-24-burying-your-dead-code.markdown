---
layout: post
title: "Burying Your Dead Code"
date: 2017-01-24 13:15:12 -0800
comments: true
author: "Lindsey Anne <lindsey.hogg@grandrounds.com>"
categories: git bisect, dead code
---
As our code evolves and changes, little bits of leftovers from refactors or changed paths can build up as dead code. It's like the plaque that builds up on your teeth over time: not really noticeable at first, but after a trip to the dentist, you feel a whole lot better.

Removing dead code from your codebase is a simple process at its core: find the dead parts, prove that they're dead, and delete them. The tricky part is in that second step, but after a little trial and error, we discovered a couple tricks to speed up the process.

##### Start small

Before digging into a gigantic repo to see what you can rip out of there, consider the impact of what you're about to do and start with just a small piece of it. Because I was investigating one of our biggest repos, I decided to tackle a single Model first rather than, say, the entire lib directory.

##### Finding the dead parts

I'm sure there are other gems out there that perform similar assessment, but our favorite gem for this is [debride by Seattle Ruby Brigade](https://github.com/seattlerb/debride). It's not 100% perfect, but it will give you a good place to start digging around. The whitelisting option is particularly helpful after your initial investigation. Another option is the brute-force method, where you comb through a file and investigate each piece one by one. If your `debride` output is suspiciously large, this might be worth a try.

##### Prove that they're dead

This is the fun part.

The specific service I was trying to clean up is one of our oldest repositories, and some of the functions that were showing up as dead ends had been sitting in there for months or even years. What's a girl to do, dig through hundreds and thousands of pull requests and commits until coming up with the right one? Oh no, we have a tool for that. It's called [git-bisect](https://git-scm.com/docs/git-bisect), and it's one of the coolest things I've learned so far about git.

`git bisect` is generally used for tracking down when a bug was introduced by using a binary search through as many commits as you'd like; you can start at the very beginning of your git history or somewhere in the middle. It checks out the repo at each commit, where you can test your issue and mark it as "good" or "bad." Rather than testing for a bug, though, I used it to `grep` for the method names that were appearing as dead from my previous step one at a time. If `grep` only gave one result (barring tests that might still exist), then nothing else was calling the method anymore and I'd mark it "bad." If it showed up with more results, I'd mark it "good." The point when searching for a bug is to find the last "good" commit before a bug was introduced; the point when using it to look for dead code is to track down when the consumer was removed.

Here's an example to illustrate the steps you'll take to do the same thing:
```
~/my_repo [master] $ git bisect start
~/my_repo [master] $ git bisect bad
~/my_repo [master] $ git bisect good ecef47e2fefc4c8ac6f3a358a4961332d24a46e3
Bisecting: 4825 revisions left to test after this (roughly 12 steps)
[0a9fc468c6efb6465c9fa96232b9f61dee01a12b] Merge pull request #5902 from my_repo/branch_5902
~/my_repo [:0a9fc46|…15375] $ git grep old_method_name
app/models/my_model.rb:  def old_method_name
~/my_repo [:0a9fc46|…15375] $ git bisect bad
Bisecting: 2412 revisions left to test after this (roughly 11 steps)
[d30229d184d5e93d39683b1f129745a211368890] Merge pull request #5346 from my_repo/branch_5346
~/my_repo [:d30229d|…15375] $ git grep old_method_name
app/models/my_model.rb:  def old_method_name
~/my_repo [:d30229d|…15375] $ git bisect bad
Bisecting: 1205 revisions left to test after this (roughly 10 steps)
[d116a5bf4d26ad1e33a099b5027883d16bbe68f2] changed a thing
~/my_repo [:d116a5b|…15375] $ git grep old_method_name
app/models/my_model.rb:  def old_method_name
~/my_repo [:d116a5b|…15375] $ git bisect bad
Bisecting: 602 revisions left to test after this (roughly 9 steps)
[ff2693d90e72798abbb8adbf57f331e661b6445b] fixed a bug
~/my_repo [:ff2693d|…15375] $ git grep old_method_name
app/helpers/my_helper.rb:    old_method_consumer = (model.try(:old_method_name) || model.try(:something_else?))
app/models/my_model.rb:  def old_method_name
~/my_repo [:ff2693d|…15375] $ git bisect good
Bisecting: 296 revisions left to test after this (roughly 8 steps)
[c6e79d2aa3185774be46473f51027d65aca6216e] Merge pull request #5040 from my_repo/branch_5040
~/my_repo [:c6e79d2|…15375] $ git grep old_method_name
app/models/my_model.rb:  def old_method_name
~/my_repo [:c6e79d2|…15375] $ git bisect bad
Bisecting: 152 revisions left to test after this (roughly 7 steps)
[74a8add547a62de49d535df07587703186057f24] Merge pull request #5017 from my_repo/branch_5017
~/my_repo [:74a8add|…15375] $ git grep old_method_name
app/helpers/my_helper.rb:    old_method_consumer = (model.try(:old_method_name) || model.try(:something_else?))
app/models/my_model.rb:  def old_method_name
~/my_repo [:74a8add|…15375] $ git bisect good
Bisecting: 68 revisions left to test after this (roughly 6 steps)
[c0d8f4c89964e95b5e5a823d0ecf36533a8c67b0] Merge pull request #5008 from my_repo/branch_5008
~/my_repo [:c0d8f4c|…15375] $ git grep old_method_name
app/models/my_model.rb:  def old_method_name
~/my_repo [:c0d8f4c|…15375] $ git bisect bad
Bisecting: 42 revisions left to test after this (roughly 5 steps)
[2a26af52b5c6234709f9816425c2cb3be7c1d3c3] changed error handling
~/my_repo [:2a26af5|…15375] $ git grep old_method_name
app/helpers/my_helper.rb:    old_method_consumer = (model.try(:old_method_name) || model.try(:something_else?))
app/models/my_model.rb:  def old_method_name
~/my_repo [:2a26af5|…15375] $ git bisect good
Bisecting: 21 revisions left to test after this (roughly 5 steps)
[37f592be9e75a68a497055109047d2e8d478cc64] Merge pull request #5038 from my_repo/branch_5038
~/my_repo [:37f592b|…15375] $ git grep old_method_name
app/helpers/my_helper.rb:    old_method_consumer = (model.try(:old_method_name) || model.try(:something_else?))
app/models/my_model.rb:  def old_method_name
~/my_repo [:37f592b|…15375] $ git bisect good
Bisecting: 10 revisions left to test after this (roughly 4 steps)
[6af2e9adc4556d153436880fb4f25e6cfa33dda0] move this thing
~/my_repo [:6af2e9a|…15375] $ git grep old_method_name
app/helpers/my_helper.rb:    old_method_consumer = (model.try(:old_method_name) || model.try(:something_else?))
app/models/my_model.rb:  def old_method_name
~/my_repo [:6af2e9a|…15375] $ git bisect good
Bisecting: 5 revisions left to test after this (roughly 3 steps)
[9ad36a544f3995e85556e9f58561d648c503ce47] added a thing
~/my_repo [:9ad36a5|…15375] $ git grep old_method_name
app/helpers/my_helper.rb:    old_method_consumer = (model.try(:old_method_name) || model.try(:something_else?))
app/models/my_model.rb:  def old_method_name
~/my_repo [:9ad36a5|…15375] $ git bisect good
Bisecting: 2 revisions left to test after this (roughly 2 steps)
[1ede562f1ab630df8b6a14dd3177413373738dca] fixed a bug
~/my_repo [:1ede562|…15375] $ git grep old_method_name
app/helpers/my_helper.rb:    old_method_consumer = (model.try(:old_method_name) || model.try(:something_else?))
app/models/my_model.rb:  def old_method_name
~/my_repo [:1ede562|…15375] $ git bisect good
Bisecting: 0 revisions left to test after this (roughly 1 step)
[8d15ccfd9d7fb0b0a6609ad87af4f5cc5566ee21] Merge pull request #5039 from my_repo/branch_5039
~/my_repo [:8d15ccf|…15375] $ git grep old_method_name
app/models/my_model.rb:  def old_method_name
~/my_repo [:8d15ccf|…15375] $ git bisect bad
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[c62e10dbe9436b9a7ad5afd37e495414591f3160] fixed another bug
~/my_repo [:c62e10d|…15375] $ git grep old_method_name
app/models/my_model.rb:  def old_method_name
~/my_repo [:c62e10d|…15375] $ git bisect bad
c62e10dbe9436b9a7ad5afd37e495414591f3160 is the first bad commit
commit c62e10dbe9436b9a7ad5afd37e495414591f3160
Author: Jane Doe <jane@Doe.com>
Date:   Thu Mar 3 12:34:56 2016 -0800

    Refactored a thing

:040000 040000 8a3d0400211d91920ea2caba83cf73f80c858b3f b39651a7ef882af4e5be2fe6f2a06a5baa4cccbe M	app
~/my_repo [:c62e10d|…15375] $
```

##### Delete them

As you can see, I finally found the commit where the diff showed the changed or removed method call; thus, I could remove the call in my current branch. After deleting all the dead code, I made sure to comment on each removed function in my pull request with the commit number where the consumer was removed (because documentation is rad). Many code reviews later, our Model was done with her teeth cleaning and felt just a little bit better.

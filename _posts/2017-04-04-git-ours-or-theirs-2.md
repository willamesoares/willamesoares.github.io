---
layout:      post
title:       "Git: Ours or Theirs? (part&nbsp;2)"
author:      Will Soares
date:        2017-04-05
categories:  tech
tags:        git
description: Tips for resolving conflicts automatically
image:       /images/posts/git/part2/mine.jpg
---

<div class="center-align">
  <img class="responsive-img" src="/images/posts/git/part2/mine.jpg">
</div>

In the [previous post](/git/git-ours-or-theirs-1.html) you were presented to a basic workflow used to automatically handle merge conflicts. It was also explained why and when you would want to do that. However, it is important to notice that, before using this method, you should be completely aware of what you are doing. In fact, whenever I get conflicts when trying to do a `pull` or a `rebase`, I use to check each one of them to see if I will be able to handle the situation quickly by using some strategy, such as using the `--ours` or `--theirs` option.

Furthermore, I also warned you that, although it might have looked a really simple strategy, there are cases in which that same strategy would provide you with different outcomes, depending on which action you are using to merge the commits.

You may remember that, in the previous scenario, we were performing a `merge` process between the branches `old-feature` and `my-new-feature`. At this point, it is important to notice that you should avoid performing this kind of process using the `merge` or `pull` methods, as explained in [this article](https://coderwall.com/p/7aymfa/please-oh-please-use-git-pull-rebase). Instead of that, you should do a `fetch` and then a `rebase` to update your local branch with a remote one. Check [this article](https://www.atlassian.com/git/tutorials/merging-vs-rebasing) for more info about the topic.

Also, before we go any further, you should remember the rule of thumb explained in the previous post:
>_The `--ours` option represents the current branch from which the merge/rebase process started before getting the conflicts, and the `--theirs` option refers to the branch where the changes are coming from._

With that, we can get into a new scenario where these options might look a little bit tricky. Let's say you now want to perform a `rebase` because there are new commits in the upstream branch. The image below represents the two branches you have and their commits before the rebase.

<div class="center-align">
  <img class="responsive-img" src="/images/posts/git/part2/example1.png">
</div>

Then, from within the `my-new-feature` you start rebasing you branch by running the command below:

```sh
$ (my-new-feature) git rebase old-feature
```

At this point, we have the key to learn how to properly use the `ours` and `theirs` options when doing a rebase.

>_Whenever you perform a rebase, the first thing Git does is to swap from your current branch to an upstream branch. Only after that, the rebase process begins._

With that in mind, it is easy to notice that the rule of thumb is still valid in this case, taking into consideration that the rebase process started in the upstream branch.

Now, you probably understand why this strategy might be tricky sometimes. You noticed that, according to the rule, the `--ours` option refers to the upstream branch with the commits from the `old-feature` branch and the `--theirs` option to the `my-new-feature` branch, as described in the image below.

<div class="center-align">
  <img class="responsive-img" src="/images/posts/git/part2/example2.png">
</div>

After the checkout, Git starts replaying all your commits on top of the commits added to the `old-feature` branch. This is one more thing Git does differently when you perform a rebase, instead of changing the HEAD pointer to the upstream branch, it updates the remote branch adding your commits to it.

Let's consider that, during the rebase, you get tons of conflicts, again, in the `index.html` file. Fortunately, you are sure that you can keep all of your changes and just ignore the changes coming from the remote branch. And you also understand that you should change your strategy a little in this scenario. Thus you go ahead and resolve the conflicts, keeping your changes, by running the command below.

```sh
$(old-feature) git checkout --theirs index.html
```

Notice, again, that the `--theirs` option here refers to your changes in `my-new-feature`.

After the rebase, you will have the following branch tree, which looks a bit different from the one you achieved in the previous post.

<div class="center-align">
  <img class="responsive-img" src="/images/posts/git/part2/example3.png">
</div>

I hope now you understand how to automate the tedious task of handling conflicts when working with Git. Also, I expect you to be careful and only use such strategies when you know what you are doing.

Thanks for reading!

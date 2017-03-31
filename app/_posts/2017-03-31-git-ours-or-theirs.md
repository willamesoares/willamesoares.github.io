---
layout:      post
title:       "Git: Ours or Theirs? (Part 1)"
author:      "Will Soares"
date:        2017-03-31
categories:  git
description: "Tips for resolving conflicts automatically"
---

<div class="center-align">
  <img class="responsive-img" src="https://cdn-images-1.medium.com/max/800/1*8oIC-d4z4EwJnOUPJuh_5A.png">
</div>

In this article, I will assume that you already have a basic understanding of Git and consequently is familiar with the process of sharing code when you are working in a team. If you do not know about that, I would suggest you to read more about those topics [here](https://backlogtool.com/git-guide/en/) and [here](https://blog.codeminer42.com/git-workflow-basics-d405746f6205) before going through this post.  

If you are here on purpose, you probably know that this article relates to a really cumbersome situation that you constantly get into when trying to share your code with coworkers, who are working in the same repository. We all know that those situations, known as conflicts, may become a lot harder or time-consuming depending on the number of changes we are trying to commit.  

As a beginner developer, I’m constantly wondering about things I could automate to increase my productivity at work. Due to that — and also because I once got a bunch of conflicts in a file — I came up with the topic for this post. So without further ado, let’s get into it.  

To begin with, let’s imagine you have to work on a new feature that you are really excited about. Thus you go ahead and create a new branch for it.

```sh
$ (old-feature) git checkout -b my-new-feature
```

After making all the changes you needed to accomplish your task, let’s say you now need to merge the old branch, from which you have created the `my-new-feature` branch, into the current one. However, before attempting to do that you notice there are new commits on that branch and the git graph now looks like this:

<div class="center-align">
  <img class="responsive-img" src="https://cdn-images-1.medium.com/max/800/1*zORfgMIo9clAqpFh8IyRqg.png">
</div>

Even though you noticed that, you think those changes on the old branch will not affect yours, so the merge command is executed within the `my-new-feature` branch.

```sh
$ (my-new-feature) git merge old-feature
```

And the output is:

```sh
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

Too bad! You’ve just got some conflicts that you should manually solve in the `index.html` file. But wait… the file is huge and there are tons of conflicts in it. You probably will take a long time to handle all of them if you do it one at a time.  

So how can we automate that? Here we are presented with two really handy options Git provides us: `ours` and `theirs` . The first option represents the current branch from which you executed the command before getting the conflicts, and the second option refers to the branch where the changes are coming from. So in the situation described above the `my-new-feature` branch would be `ours` and the `old-feature` branch would be `theirs`.

<div class="center-align">
  <img class="responsive-img" src="https://cdn-images-1.medium.com/max/800/1*_OtZLrRG59_aGmbwTQPBUA.png">
</div>

In order to better understand when you would want to use any of those options, let’s imagine that you opened the `index.html` file and you noticed that all the changes coming from the `old-feature` branch are not correct and you can simply ignore them. With that, you want to keep only the changes from your branch, which are referred to by the `ours` option. So instead of handling them one line at a time you can simply run the command below.

```sh
$ (my-new-feature) git checkout --ours index.html
```

After that, you just need to explicitly tell Git that you solved the conflicts in that file by adding it to the Index, as shown in the command below.

```sh
$ (my-new-feature) git add index.html
```

Similarly, if you would want to keep all the changes coming from the `old-feature` branch and ignore your changes, you would just have to use the option `theirs` instead (in situations like the one described above, of course).  

After the merge process is successfully finished we would have a branches graph like this:

<div class="center-align">
  <img class="responsive-img" src="https://cdn-images-1.medium.com/max/800/1*SrgDOVkBwLX3haHzJ7so1w.png">
</div>

With that you have used a strategy to resolve conflicts that may happen when doing a `git merge`. This allows you to save a considerable amount of time when you face problems like this.

It is important to notice that you must be careful when resolving conflicts this way. One must be sure about which changes should be kept and which ones should be ignored before doing it automatically. Besides, those options can be a little bit tricky in different situations. The one described here is specific to a merge process and should not be mistaken for cases when you are doing a rebase , for instance. But that is going to be discussed later in another post. Stay tuned!

...

In the meantime, if you want to learn more about the options and the other commands discussed you can take a look [here](https://git-scm.com/book/tr/v2/Git-Tools-Advanced-Merging).  

If you have any questions or comments, let me know!  

Thanks a bunch for reading!

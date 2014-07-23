---
layout: post
title: "Git: Moving History"
date: 2014-07-23 20:37:24 +0300
comments: true
categories: [Coding, Shell]
---

The title of this post may make it sound like it's going to be about how git changed the face of software development and entered us into a
new era of coding collaboration. No. This post is about how to copy a subtree from one git repository to another, while keeping its history.

Let's say you've been working on a component in an incubator-style repository, and it's time to move it to the main project's repository.
Simply copying the code would destroy the valuable history in the target repo, so that should be avoided. There are also lots of other
components in the incubator repo that you don't want to move.

## Step 1: Export your component to a temporary repository

You can do this using `git filer-branch`. *Warning:* this will reduce your local copy of the incubator repository to *just* your component.
Either clone it locally to a different path or be prepared to clone it again from the remote.

``` console
incubator$ git filter-branch --subdirectory-filter components/my-component -- --all
```

Now your repository has been reduced to just the contents **and history** of `components/my-component`.

## Step 2: Create a patch for the entire history of your component

Use `git format-patch` to export all of your commits to a patch file. Props to [@rombert] for this idea.

``` console
incubator$ git format-patch --stdout --root $(git rev-list HEAD | tail -n 1) HEAD > my-component.patch
```

I feel like this command could use some explaining. First off, we're telling `format-patch` to output everything to `--stdout`. Otherwise,
it would create one patch file per commit, which can get pretty clumsy if there are lots of them. Second, we're passing the output of
`git rev-list HEAD | tail -n 1` for the `--root` parameter. The enclosed command will find find the `sha1` of the very first commit, while
the `--root` parameter will tell `format-patch` to include that commit, not start from it. Lastly, `HEAD` is the target ref, which is
basically the most recent commit.

## Step 3: Apply the patch

Now it's time to add your component to the main project. Of course, you don't want to add your component to the root of the repository
(which is what all the paths in the patch are relative to). Thankfully, git supports prepending a path to all filenames in a patch.

``` console
main-project$ git am --directory components/my-component my-component.patch
```

Now your main project contains your component and all of its history. Some notes:

1. Only one branch will be copied. If your component development happened on multiple branches, that will be lost.
2. No tags will be copied to the main repository, you will need to tag your commits manually.
3. Commit IDs will change, so you will not be able to [copy your tags automatically][git-copy-tags].

Pretty neat.

[@rombert]: https://twitter.com/rombert
[git-copy-tags]: https://github.com/foghina/git-copy-tags

# A Guide to Understanding the Nuances of Git Merge and Rebase

## Introduction

I've seen many developers **avoid using rebase** due to its complexity. I've also noticed that some teams even forbid rebase in their projects altogether.

In this guide, we will walk through the basics of managing code changes in Git using both merge and rebase strategies. This tutorial aims to clarify these often misunderstood commands with a hands-on demonstration.

I may be too verbose in walking you through these steps one by one, but I suggest you start from the [Recap](#recap) section, or jump straight to the [Key Takeaways](#key-takeaways-merge-or-rebase) section.

## Table of Contents

- [Introduction](#introduction)
- [Scenario Setup](#scenario-setup)
  - [Initialise a Git Repository and Create a Starting File](#initialise-a-git-repository-and-create-a-starting-file)
  - [Make Initial Commits A and B](#make-initial-commits-a-and-b)
  - [Bseline Commit Graph](#bseline-commit-graph)
  - [Create a Feature Branch](#create-a-feature-branch)
  - [Commit C](#commit-c)
  - [Commit D (Developing on Feature Branch)](#commit-d-developing-on-feature-branch)
- [Initial Git Commit Graph](#initial-git-commit-graph)
- [Decision Time: Merging vs. Rebasing](#decision-time-merging-vs-rebasing)
- [Option 1: Merge feature into main](#option-1-merge-feature-into-main)
  - [Merge](#merge)
  - [Merge Conflict](#merge-conflict)
  - [Graph After Merge](#graph-after-merge)
- [Option 2: Rebase feature onto main](#option-2-rebase-feature-onto-main)
  - [Rebase](#rebase)
  - [Rebase Conflict](#rebase-conflict)
  - [Graph After Rebase](#graph-after-rebase)
- [Recap](#recap)
  - [Commit History](#commit-history)
  - [Commit Graph of Taking Each Option](#commit-graph-of-taking-each-option)
- [Key Takeaways: Merge or Rebase?](#key-takeaways-merge-or-rebase)

## Scenario Setup

### Initialise a Git Repository and Create a Starting File

First, create a new directory and initialise a Git repository:

```sh
mkdir git-nuance-demo-merge && \
cd git-nuance-demo-merge && \
git init && \
git branch -M main
```

Next, create a text file named `example.txt`:

 ```sh
touch example.txt
 ```

### Make Initial Commits `A` and `B`

Now, add a first line to `example.txt` and commit it as Commit `A`:

```diff
+Initial content
```

<!--
<img alt="" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-1.png" width="500" />
-->

Proceed by adding a second line and commit it as Commit `B`:

```diff
Initial content
+Added by leader
```

<!--
<img alt="change" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-2.png" width="500" />
-->

### Bseline Commit Graph

Now we can visualize the project's **baseline** with this commit graph:

```css
A---B [main]
```

### Create a Feature Branch

Assuming developer Kev needs to add a new feature, he branches off from `main`:

```sh
git checkout main && \
git checkout -b feature
```

### Commit `C`

While Kev works on the feature branch, another developer, Dash, updates the `main` branch. Dash modifies the **second** line in Commit `C`:

```sh
git checkout main
```

```diff
Initial content
-Added by leader
+Modified by Dash
```

<!--
<img alt="change" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-3.png" width="500" />
-->

### Commit `D` (Developing on Feature Branch)

Simultaneously, Kev modifies the **same** line on his branch in Commit `D`:

```sh
git checkout feature
```

```diff
Initial content
-Added by leader
+Modified by Kev
```

<!--
<img alt="change" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image.png" width="500" />
-->

## Initial Git Commit Graph

The commit graph is now:

```css
A---B---C [main]
     \
      D [feature]
```

And check their Git history details by running `git log main` and `git log feature`:

> The commit IDs (hashes) will be different if you try following my steps from the beginning on your device.

- `main`

<!--
  <img alt="Git Logs" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-4.png" width="500" />
-->

```console
/workspaces/git-nuance-demo-merge (main) $ git log main

commit ac466bea2d13762608660e27798ba08b840529db (HEAD -> main)
Author: Dash <dash@test.com>

    C

commit fd6ecc11625c41c06e150015254b16a620c1cd44
Author: Leader <leader@test.com>

    B

commit 9d00689a138f8d1fd6b7d370bae29bcfb27fec19
Author: Leader <leader@test.com>

    A

```

- `feature`

<!--
  <img alt="Git Logs" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-5.png" width="500" />
-->

```console
/workspaces/git-nuance-demo-merge (main) $ git log feature

commit 58beb1e4f1d47f9bf1364be906a2e22b5280be1d (HEAD -> feature)
Author: Kev <kev@test.com>

    D

commit fd6ecc11625c41c06e150015254b16a620c1cd44
Author: Leader <leader@test.com>

    B

commit 9d00689a138f8d1fd6b7d370bae29bcfb27fec19
Author: Leader <leader@test.com>

    A

```

## Decision Time: Merging vs. Rebasing

Imagine Kev needs to release the features in branch `feature`, he faces a decision: to merge or rebase his feature branch onto the updated main branch.:

- `Merge`: merge `feature` into `main` and release `main` to production environment.
- `Rebase`: Rebase `feature` onto `main`, and release `feature`.

We all know that Kev and Dash both edited the **second** line, meaning a **conflict** is definitely going to happen in both options.

Next, we're going to apply both options and examine the different consequences of each.

> Before continuing, copy the initial status into two separate folders so that we can apply each option in its own folder:
>
> ```sh
> cd .. && \
> cp -r git-nuance-demo-merge git-nuance-demo-rebase
>  ```

## Option 1: Merge `feature` into `main`

### Merge

Merging results in a new commit on `main` that combines the changes from both branches:

```sh
cd git-nuance-demo-merge
```

```sh
git checkout main
```

```sh
git merge feature
```

<img alt="merge feature into main" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/merge-c.gif" width="600" />

### Merge Conflict

This approach creates a merge conflict that Kev resolves by favouring his changes:

```txt
Initial content
<<<<<<< HEAD
Modified by Dash
=======
Modified by Kev
>>>>>>> feature
```

Pay attention to the terminal console:

```console
/workspaces/git-nuance-demo-merge (main) $ git merge feature
Auto-merging example.txt
CONFLICT (content): Merge conflict in example.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Check this out: "**fix conflicts and then commit the result**"

It instructs us to both **fix** and **commit** so that the conflicts can be resolved. This means a **new commit** is needed when resolving merge conflicts.

Resolve it by selecting "**Accept Incoming Change**", which in this case means accepting the changes from the `feature` branch.

<img alt="fix git merge conflict and commit" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/merge-c-2.gif" width="600" />

If you check the log by running `git log main`, you'll notice the new merge commit in the logs saying "**Merge branch..**.":

<!--
<img alt="merge" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-7.png" width="500" />
-->

```console
/workspaces/git-nuance-demo-merge (main) $ git log main

commit 4d6ac5eb811be1a6130c6bbcde215946c618dd95 (HEAD -> main)
Merge: ac466be 58beb1e
Author: Kev <kev@test.com>

    Merge branch 'feature'

commit 58beb1e4f1d47f9bf1364be906a2e22b5280be1d (feature)
Author: Kev <kev@test.com>

    D

commit ac466bea2d13762608660e27798ba08b840529db
Author: Dash <dash@test.com>

    C

commit fd6ecc11625c41c06e150015254b16a620c1cd44
Author: Leader <leader@test.com>

    B

...
```

Wondering what the change is in this new commit?

Run `git diff 4d6ac5e~ 4d6ac5e` (4d6ac5e is the commit ID (hash))

<!--
<img alt="change of merge commit" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-8.png" width="500" />
-->

```diff
/workspaces/git-nuance-demo-merge (main) $ git diff 4d6ac5e~ 4d6ac5e
diff --git a/example.txt b/example.txt
index 22ad65d..d49a266 100644
--- a/example.txt
+++ b/example.txt
@@ -1,2 +1,2 @@
 Initial content
-Modified by Dash
+Modified by Kev
```

If you take a deeper look at the details of this new commit, pay special attention to the line:

**Merge: `ac466be` `58beb1e`**

What does it mean?

This **merge** commit has two **parents**, pointing to commit `C` and commit `D`:

<img alt="merge commit parents" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-10.png" width="500" />

### Graph After Merge

After resolving the conflict, the commit graph looks like:

```css
A---B---C---M [main]
     \       /
      D [feature]
```

## Option 2: Rebase `feature` onto `main`

Before continuing with the rebase process, take a look at the `feature` branch's logs at this point:

<!--
<img alt="Git Logs" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-6.png" width="500" />
-->

```console
/workspaces/git-nuance-demo-merge (main) $ git log feature

commit 58beb1e4f1d47f9bf1364be906a2e22b5280be1d (Head -> feature)
Author: Kev <kev@test.com>

    D
...
```

The commit `D`'s ID (hash) is `58beb1e....`

The change in this commit is:

<!--
<img alt="change" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-13.png" width="500" />
-->

```diff
/workspaces/git-nuance-demo-merge (main) $ git diff 58beb1e~ 58beb1e
diff --git a/example.txt b/example.txt
index a53a0e2..d49a266 100644
--- a/example.txt
+++ b/example.txt
@@ -1,2 +1,2 @@
 Initial content
-Added by leader
+Modified by Kev
```

The commit ID and changes will be altered after the rebase; we'll see why.

### Rebase

Let's now start the **rebase** process.

Alternatively to **option 1**, rebasing involves integrating the changes from `main` directly into the `feature` branch history.

```sh
cd git-nuance-demo-rebase
```

```sh
git checkout feature
```

```sh
git rebase main
```

<img alt="rebase feature onto main" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/rebase-c.gif" width="600"/>

### Rebase Conflict

It creates a merge conflict that is almost identical to the one in **option 1**:

```txt
Initial content
<<<<<<< HEAD
Modified by Dash
=======
Modified by Kev
>>>>>>> 58beb1e (D)
```

But what's in the terminal console is different:

```console
/workspaces/git-nuance-demo-rebase (feature) $ git rebase main
Auto-merging example.txt
CONFLICT (content): Merge conflict in example.txt
error: could not apply 58beb1e... D
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply 58beb1e... D
```

It instructs us to "**Resolve all conflicts manually**", and continue with `git rebase --continue`.

Alright, let's follow the hint to resolve the conflict (also by selecting "Accept Incoming Change").

```sh
git add example.txt && \
git rebase --continue
```

<img alt="fix git rebase conflict and commit" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/rebase-c-2.gif" width="600"/>

Take a look at the content after running `git rebase --continue`:

```txt
D

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# interactive rebase in progress; onto 05cfdd2
# Last command done (1 command done):
#    pick 58beb1e D
# No commands remaining.
# You are currently rebasing branch 'feature' on '05cfdd2'.
#
# Changes to be committed:
# modified:   example.txt
#
```

Git offers us an editor to edit the commit message of `D`.

You can rewrite commit `D` by editing the message (lines starting with "#" will be ignored).

I made no changes and just closed the commit editor, so the rebase process proceeded with the **rewriting** of commit `D`.

Now that the rebase process has finished, take a look at the commit history of the `feature` branch:

```console
/workspaces/git-nuance-demo-rebase (feature) $ git log feature

commit 047c3480da20d9dd6c530d3698b4e047f1f1d9d9 (HEAD -> feature)
Author: Kev <kev@test.com>

    D

commit ac466bea2d13762608660e27798ba08b840529db (main)
Author: Dash <dash@test.com>

    C

...
```

<img alt="Git Logs" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-11.png" width="500" />

Pay special attention to the commit ID of the **new** `D`.

It's now `047c348...`. Remember what it was before the rebase? It was `58beb1e...`.

And the change in this commit is:

<!--
<img alt="change" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-9.png" width="500" />
-->

```diff
/workspaces/git-nuance-demo-rebase (feature) $ git diff 047c348~ 047c348
diff --git a/example.txt b/example.txt
index 22ad65d..d49a266 100644
--- a/example.txt
+++ b/example.txt
@@ -1,2 +1,2 @@
 Initial content
-Modified by Dash
+Modified by Kev
```

See? The change is also different from the previous commit `D` because the commit `D` was **rewritten** in the rebase process.

### Graph After Rebase

Rebase modifies the original commit `D` on the feature branch:

```css
A---B---C [main]
         \
          D' [feature]
```

In this case, we performed a **history rewrite**, causing the original commit `D` to disappear and the new commit `D'` to emerge.

## Recap

### Commit History

Now let's recap the primary steps and Git histories in both options we covered.

<table>
  <thead>
    <tr>
      <td>Option 1: Merge</td>
      <td>Option 2: Rebase</td>
    </tr>
  </thead>
  <tbody>

<tr>

<td>

<table>
  <thead>
    <tr>
      <td>Commit</td>
      <td>Diff</td>
      <td>Author</td>
    </tr>
  </thead>
  <tbody>

<tr>

<td>
Merge
</td>

<td>

```diff
-Modified by Dash
+Modified by Kev
```

</td>

<td>
Kev
</td>

</tr>

<tr>

<td>
D
</td>

<td>

```diff
-Add by leader
+Modified by Kev
```

</td>

<td>
Kev
</td>

</tr>

<tr>

<td>
C
</td>

<td>

```diff
-Add by leader
+Modified by Dash
```

</td>

<td>
Dash
</td>

</tr>

<tr>
<td>B</td>
<td>

```diff
+Add by leader
```

</td>
<td>Leader</td>
</tr>

<tr>
  <td>A</td><td>...</td><td>Leader</td>
</tr>

  </tbody>
</table>

</td>

<td>

<table>
  <thead>
    <tr>
      <td>Commit</td>
      <td>Diff</td>
      <td>Author</td>
    </tr>
  </thead>
  <tbody>

<tr>

<td>
D'
</td>

<td>

```diff
-Modified by Dash
+Modified by Kev
```

</td>

<td>
Kev
</td>

</tr>

<tr>

<td>
D (actually disappeared)
</td>

<td>

```diff
-Add by leader
+Modified by Kev
```

</td>

<td>
Kev
</td>

</tr>

<tr>

<td>
C
</td>

<td>

```diff
-Add by leader
+Modified by Dash
```

</td>

<td>
Dash
</td>

</tr>

<tr>
<td>B</td>
<td>

```diff
+Add by leader
```

</td>
<td>Leader</td>
</tr>

<tr>
  <td>A</td><td>...</td><td>Leader</td>
</tr>

  </tbody>
</table>

</td>

</tr>

  </tbody>
</table>

> You can also view the histories in my demo repositories:
>
> - Option 1: [Merge](https://github.com/graezykev/git-nuance-demo-merge/commits/main/)
> - Option 2: [Rebase](https://github.com/graezykev/git-nuance-demo-rebase/commits/feature)

### Commit Graph of Taking Each Option

<table>
<thead>
<tr>
<td>Merge</td>
<td>Rebase</td>
</tr>
</thead>
<tbody>

<tr>

<td>

```css
A---B---C---M [main]
     \       /
      D [feature]
```

</td>

<td>

```css
A---B---C [main]
         \
          D' [feature]
```

</td>

</tr>

</tbody>
</table>

The obvious difference is that there are **five** commits (`A-B-C-D-M`) after a **merge**, and **four** (`A-B-C-D'`) after a **rebase** (D disappears because it is rewritten).

### Git Rebase can be Dangerous

**Merge** preserves the exact historical sequence of changes and is generally easier for beginners to understand and handle, especially in a collaborative environment.

In contrast, **Rebase** rewrites the commit history to make it appear as if your changes were created on top of the latest remote commits.

This can result in a cleaner, more linear commit history but can be confusing because it alters the commit history. This might be trickier in a collaborative project unless all contributors are comfortable with Git.

This is typical evidence of why **Git rebase can be dangerous**:

In more complex, real-world scenarios, there is a risk that you could rewrite, overwrite, or delete another developer's commit from the history, preventing your teammates from finding their previously made commits.

There are some basic principles in using rebase like "**Don’t rebase a branch that’s been published remotely**" and "**Create a backup branch from the tip of the branch you’re about to rebase**" etc.

There are some basic principles for using rebase, such as "**Don’t rebase a branch that’s been published remotely**" and "**Create a backup branch from the tip of the branch you’re about to rebase**".

Choosing between merge and rebase based on your project's needs is a broad topic that is not comprehensively covered within the scope of this post. For more detailed information, I recommend reading "[Differences Between Git Merge and Rebase — and Why You Should Care](https://blog.git-init.com/differences-between-git-merge-and-rebase-and-why-you-should-care/#conclusion)".

## Key Takeaways: Merge or Rebase?

Both strategies have their merits:

- `Merge`: You create a **new commit** that shows how conflicts were resolved, allowing your teammates to clearly see the resolution process.

- `Rebase`: You **modify existing commits**. When resolving conflicts during a rebase, you may alter code or commit messages that others have previously made. This can disrupt collaborative work if not used carefully.

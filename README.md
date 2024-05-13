# Git Merge vs. Rebase: The Nuance

Explore the intricate dance between merging and rebasing in Git, clarify these often misunderstood commands with a hands-on demonstration.

## Introduction

I may be too verbose to list these steps one by one, but I suggest you start from the [Recap](#recap) part, or jump straightly to the [conclusion](#conclusion-nuance).

## Table of Contents

## Scenario Setup

### Initialise a Git Repository & Create a Starting File

Initiate a Git repository, as well as a `main` branch.

```sh
mkdir git-nuance-demo-merge && \
cd git-nuance-demo-merge && \
git init && \
git branch -M main
```

 ```sh
touch example.txt
 ```

### Make Initial Commits `A` and `B`

Input a first line into `example.txt`

```diff
+Initial content
```

<!--
<img alt="" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-1.png" width="500" />
-->

And then make a commit of 'A'.

Input a second line into `example.txt`

```diff
Initial content
+Added by leader
```

<!--
<img alt="change" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-2.png" width="500" />
-->

And then make a commit of 'B'.

### Bseline Commit Graph

Here's the commit graph by far, imagine it as the project **baseline**.

```css
A---B [main]
```

### Create a Feature Branch

Let's say one of the developers Kev needs to develop a new feature, he creates a new branch `feature` based on `main`.

```sh
git checkout main && \
git checkout -b feature
```

### Commit `C`

During Kev's development, another developer Dash edited branch `main` first, with a commit `C` (maybe by merging his own feature).

```sh
git checkout main
```

In this commit `C` he **modified the second line**.

```diff
Initial content
-Added by leader
+Modified by Dash
```

<!--
<img alt="change" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-3.png" width="500" />
-->

### Commit `D` (Developing on Feature Branch)

Return to Kev's branch `feature`, he also makes some changes to the **second line**.

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

And he makes a commit as `D`.

## Initial Git Commit Graph

Simply put Kev's and Dash's commits above into a commit graph.

```css
A---B---C [main]
     \
      D [feature]
```

Here are the **Git histories** on branch `main` and `feature` respectively.

- Git Logs on `main`

  <img alt="Git Logs" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-4.png" width="500" />

- Git Logs on `feature`

  <img alt="Git Logs" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-5.png" width="500" />

## Decision Time: Merging vs. Rebasing

Imagine Kev needs to release the features in branch `feature`, he has 2 options:

- `Merge`: merge `feature` into `main` and release `main` to production environment.
- `Rebase`: Rebase `feature` onto `main`, and release `feature`.

We all know right now Kev and Dash both edited line 2, meaning a **conflict** is definitely going to happen in both options.

Next, we're going to simulate making the 2 options respectively, and find out what are the different consequences of them.

## Option 1: Merge `feature` into `main`

```sh
git checkout main
```

```sh
git merge feature
```

<img alt="merge feature into main" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/merge-c.gif" width="600" />

Here's the conflict (in line 2).

```txt
Initial content
<<<<<<< HEAD
Modified by Dash
=======
Modified by Kev
>>>>>>> feature
```

And take notes on the console.

```console
/workspaces/git-nuance-demo-merge (main) $ git merge feature
Auto-merging example.txt
CONFLICT (content): Merge conflict in example.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Check this one out: "**fix conflicts and then commit the result**"

It tells us to both **Fix** and **Commit** so that the conflicts can be resolved.

Let's do it by "Accept Incoming Change", which in this case means accepting branch `feature`'s change.

<img alt="fix git merge conflict and commit" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/merge-c-2.gif" width="600" />

Do you notice the **new** commit in the Git Logs of `main`?

<img alt="merge" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-7.png" width="500" />

The details of the new commit (**Merge Commit**).

<img alt="change of merge commit" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-8.png" width="500" />

```diff
Initial content
-Modified by Dash
+Modified by Kev
```

If you take a deeper look at the new commit, it has 2 **parents**, in this case, commit `C` and commit `D`

<img alt="merge commit parents" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-10.png" width="500" />

### Graph After Merge

```css
A---B---C---M [main]
     \       /
      D [feature]
```

> `M` represents the new (Merge) commit (after fixing the conflict).

## Option 2: Rebase `feature` onto `main`

> Before continuing the rebase process, **Re-do** the **Scenario Setup** with another Git repo

the **Scenario Setup**, our [Initial Git Commit Graph](#initial-git-commit-graph) looks as this.

```css
A---B---C [main]
     \
      D [feature]
```

Take a look at branch `feature`'s logs at this point.

<img alt="Git Logs" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-6.png" width="500" />

The commit `D`'s commit ID (hash) is `58beb1e...`.

And the changes in this commit is:

<!--
<img alt="change" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-13.png" width="500" />
-->

```diff
Initial content
-Added by leader
+Modified by Kev
```

The commit ID and changes are going to be changed after the rebase, we'll see why.

Now let's start the **Rebase**.

Go to `feature`.

```sh
git checkout feature
```

Rebase it onto `main`.

```sh
git rebase main
```

<img alt="rebase feature onto main" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/rebase-c.gif" width="600"/>

Here's the conflict (also in line 2)

```txt
Initial content
<<<<<<< HEAD
Modified by Dash
=======
Modified by Kev
>>>>>>> 58beb1e (D)
```

The conflict looks **identical** to what we did in the last option (Merge) doesn't it? But the terminal console is different.

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

It tells us to **Resolve all conflicts manually**, and continue with `git rebase --continue`.

OK let's follow the guide to resolve the conflict (also by "Accept Incoming Change").

<img alt="fix git rebase conflict and commit" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/rebase-c-2.gif" width="600"/>

Take some insight into the content after I run `git rebase --continue`.

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

Here Git offers me an editor to edit the commit message of `D`, you can rewrite the commit `D` by editing the message (lines starting with `#` will be ignored).

I made no change and just close the commit editor, the rebase process was continued with the "**rewritten**" of commit `D`.

Now the rebase has finished, take a look at the commit history of branch `feature`.

<img alt="Git Logs" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-11.png" width="500" />

Pay special attention to the commit ID of `D` (new `D`).

It's now `047c348...`, remember what was it before the rebase? It was `58beb1e...`.

And the changes of this commit.

<!--
<img alt="change" src="https://raw.githubusercontent.com/graezykev/git-nuance-merge-rebase/main/image-9.png" width="500" />
-->

```diff
Initial content
-Modified by Dash
+Modified by Kev
```

See? The changes are also different to the previous commit `D`, because the commit `D` was **rewritten** in the rebase process.

> Actually the commit message could also have been modified as well, in the commit editor we saw after ``git rebase --continue`.

### Graph After Rebase

```css
A---B---C [main]
         \
          D' [feature]
```

In this case, we made a "**History Rewrite**", after which the commit `D` disappears and the new commit `D'` comes out.

## Recap

### Commit History

Now let's recap the primary steps and Git histories in both options we made.

> You can also see the histories in my demo repositories:
>
> - Option 1: [Merge](https://github.com/graezykev/git-nuance-demo-merge/commits/main/)
> - Option 2: [Rebase](https://github.com/graezykev/git-nuance-demo-rebase/commits/feature)

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

### Commit Graph After each Option

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

There are `5` commits (`A-B-C-D-M`) after "**Merge**", and `4` (`A-B-C-D'`) after "**Rebase**" (`D` disappeared because of rewritten).

The **Merge** option preserves the exact history of changes and is generally easier for beginners to understand and handle, especially in a collaborative environment.

The **Rebase** option rewrites the commit history to make it look as if you've created your changes on top of the latest remote commits. This can make the commit history cleaner and linear but can be confusing because it alters commit history. This might be trickier in a collaborative project unless all contributors are comfortable with Git.

### Git Rebase can be Dangerous

This is typical evidence of **why Git Rebase can be Dangerous**:

In a more complicated, more real-world senario, there're risks that you can rewrite/overwrite/delete another developer's commit from the history, meaning your teammates may not able to find their previously made commit(s).

I've seen lots of developers are prone to **avoid using rebase**, or setting up princeples like "**Don’t rebase a branch that’s been published remotely**".

Choosing between merge and rebase based on your project’s need is a very broaden topic which is not whitin my topic scope in this post, I find a better learning material for your information: [Differences Between Git Merge and Rebase — and Why You Should Care](https://blog.git-init.com/differences-between-git-merge-and-rebase-and-why-you-should-care/#conclusion).

## Conclusion: Nuance

After fixing the conflicts:

- `Merge`: You need to make a **new commit** indicating how you solved the conflicts, from which your teammates can straightforwardly see how you did that.

- `Rebase`: You **modify the existent commit**, in this modified commit your teammates may see how you solve the conflicts.

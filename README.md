## Scenario Setup

Initiate a Git repository, as well as a `main` branch.

```sh
mkdir git-nuance-merge && \
cd git-nuance-merge && \
git init && \
git branch -M main
```

Create a file.

 ```sh
touch example.txt
```

### Commit `A`

Input a first line into `example.txt`

```diff
+Initial content
```

And then make a commit of 'A'.

<img alt="" src="image-1.png" width="500" />

### Commit `B`

Input a second line into `example.txt`

```diff
Initial content
+Added by main
```

And then make a commit of 'B'.

<img alt="" src="image-2.png" width="500" />

### Bseline Commit Graph

Here's the commit graph now, imagine it as the project baseline.

```css
A---B [main]
```

### Commit `D`

Let's say one of the developers (or myself) needs to develop a new feature, he creates a new branch `feature` based on `main`.

```sh
git checkout -b feature
```

And makes some changes to the second line.

```diff
Initial content
-Added by main
+Modified by feature
```

<img alt="" src="image-3.png" width="500" />

### Commit `C`

During his development, another developer (or myself) merged his feature to the branch `main`, with a commit `C` .

```sh
git checkout main
```

In this commit he also made some changes to the second line.

```diff
Initial content
-Added by main
+Modified by main
```

<img alt="" src="image.png" width="500" />

### Initial Git Commit Graph

Simply put the two guys's commits above into this graph:

```css
A---B---C [main]
     \
      D [feature]
```

Here are the **Git Histories** on branch `main` and `feature` respectively.

- Git Logs on `main`

  <img alt="" src="image-4.png" width="200" />

- Git Logs on `feature`

  <img alt="" src="image-5.png" width="200" />

Imagine we need to release the features in brach `feature`, we have 2 options:

- `Merge`: merge `feature` into `main` and release `main` to production environment.
- `Rebase`: Rebase `feature` onto `main`, and release.

Next, we're going to make the 2 options respectively, and find out what are the different consequences of them.

## Option 1: Merge `feature` into `main`

```sh
git checkout main
```

```sh
git merge feature
```

<img alt="" src="merge-c.gif" width="600" />

Here's the confict (in line 2).

<img alt="" src="image-6.png" width="600" />

```txt
Initial content
<<<<<<< HEAD
Modified by main
=======
Modified by feature
>>>>>>> feature
```

And take notes on the console.

```console
➜  git-nuance-merge git:(main) git merge feature
Auto-merging example.txt
CONFLICT (content): Merge conflict in example.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Check it out: **fix conflicts and then commit the result.**

It tells us to both **Fix** and **Commit** so that the conficts can be resolved.

Let's do it by "Accept Incoming Chage", which in this case means accepting `feature` brach's change.

<img alt="fix git merge conflict and commit" src="merge-c-2.gif" width="600" />

Did you notice the new commit in the Git Logs of `main`?

<img alt="merge" src="image-7.png" width="250" />

The details of the new commit (**Merge Commit**)

<img alt="merge" src="image-8.png" width="250" />

```diff
Initial content
-Modified by main
+Modified by feature
```

### Graph After Merge

```css
A---B---C---M [main]
     \       /
      D [feature]
```

> `M` represents the new (Merge) commit (after fixing conflict)

## Option 2: Rebase `feature` onto `main`

> Before continuing the rebase process, **Re-do** the Scenario Setup in another Git repo

This is our initial commit graph.

```css
A---B---C [main]
     \
      D [feature]
```

Take a look at the commit `D`'s commit ID (hash) `89ccb71` on brach `feature`.

<img alt="" src="image-10.png" width="250" />

And the changes in this commit.

```diff
Initial content
-Added by main
+Modified by feature
```

The commit ID and changes are going to be changed after rebase, we'll see why.

Now let's start the **Rebase**.

Go to `feature`.

```sh
git checkout feature
```

Rebase it onto `main`.

```sh
git rebase base
```

<img alt="" src="rebase-c.gif" width="600"/>

Here's the confict

<img alt="" src="image-9.png" width="600"/>

```txt
Initial content
<<<<<<< HEAD
Modified by main
=======
Modified by feature
>>>>>>> 89ccb71 (D)
```

The conflict looks **identical** to what we did in the last option (Merge) doesn't it? But the terminal console is different.

```console
➜  git-nuance-rebase git:(feature) git rebase main
Auto-merging example.txt
CONFLICT (content): Merge conflict in example.txt
error: could not apply 89ccb71... D
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply 89ccb71... D
```

It tells us to **Resolve all conflicts manually**, and continue with `git rebase --continue`.

OK let's follow the guide to resolve the conflict (also by "Accept Incoming Chage").

<img alt="fix git rebase conflict and commit" src="rebase-c-2.gif" width="600"/>

Take some insight into the content after I run `git rebase --continue`.

```txt
D

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# interactive rebase in progress; onto 755ac4a
# Last command done (1 command done):
#    pick 89ccb71 D
# No commands remaining.
# You are currently rebasing branch 'feature' on '755ac4a'.
#
# Changes to be committed:
#       modified:   example.txt
#
```

Here Git offers me a editor to edit the commit message of `D`.

I can make no chage and just leave it.

Now the rebase has finished, take a look at the commit history of `feature`.

<img alt="" src="image-11.png" width="250" />

And the commit ID of `D`.

<img alt="" src="image-12.png" width="250" />

It's `d512c2f`, remember what was it before rebase? It was `89ccb71`.

And the changes of this commit.

```diff
Initial content
-Modified by main
+Modified by feature
```

See? The changes are also different to the previous commit `D`, because the commit `D` was **rewritten** in rebase.

> Actually the commit message could also has been modify as well, in the commit editor we saw after ``git rebase --continue`.

### Graph After Rebase

```css
A---B---C [main]
         \
          D' [feature]
```

In this case, we did a "**History Rewrite**", after which `D` disappears and the new `D'` comes out.

This is a typical evidence of **why Git Rebase can be Dangerous**.

## Conclusion: Nuance

After fix the conflicts:

- `Merge`: You need to make a **new commit** indicating how you solve the conflicts, from which your teammates can straightforwardly see how you did that.

- `Rebase`: You **modify the existent commit**, in this modified commit your teammates may see how you solve the conflicts.
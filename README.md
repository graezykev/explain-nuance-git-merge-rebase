I skip the initial of the Git repository because that is sort of petty, I just assume we have a repo with a `main` brach and without any commit.

## Scenario Setup

Creat a file `example.txt`.

### Commit `A`

input the first line into `example.txt`

```diff
+Initial content
```

And then make a commit of 'A'.

![alt text](image-1.png)

### Commit `B`

input the second line into `example.txt`

```diff
Initial content
+Added by main
```

And then make a commit of 'B'.

![alt text](image-2.png)

### Initial Commit Graph

Here's the commit graph by now.

```css
A---B [main]
```

### Commit `D`

Create a new branch `feature` based on `main`.

```sh
git checkout -b feature
```

Make some change to the second line.

```diff
Initial content
-Added by main
+Modified by feature
```

![alt text](image-3.png)

### Commit `C`

Get back to branch `main`.

```sh
git checkout main
```

And Make some change to the second line too.

```diff
Initial content
-Added by main
+Modified by main
```

![alt text](image.png)

### Initial Git Commit Status

Here is what we've done by now on branch `main` and `feature`.

- Git Logs on `main`

  <img alt="" src="image-4.png" width="200" />

- Git Logs on `feature`

  <img alt="" src="image-5.png" width="200" />

Simply put them into this graph:

```css
A---B---C [main]
     \
      D [feature]
```

Imagine we need to release the features in brach `feature`, we have 2 options:

- `Merge`: merge `feature` into `main` and release `main` to production environment.
- `Rebase`: Rebase `feature` onto `main`, and release.

Next, we're going to make the 2 options respectively, and find out what are the difference consequences of them.

## Option 1: Merge

```sh
git checkout main && \
git merge feature
```

![alt text](merge-c.gif)

Here's the confict

![alt text](image-6.png)

```txt
Initial content
<<<<<<< HEAD
Modified by main
=======
Modified by feature
>>>>>>> feature
```

And take notes on the console

```console
➜  git-nuance-merge git:(main) git merge feature
Auto-merging example.txt
CONFLICT (content): Merge conflict in example.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Check it out: **fix conflicts and then commit the result.**

It tells us to both **Fix** and **Commit** so that the conficts can be resolved.

Let's do it By "Accept Incoming Chage", which in this case means accepting `feature` brach's change.

![fix git conflict and commit](merge-c-2.gif)

Did you notice the new commit in the Git Logs?

The Git Logs on `main`

<img alt="merge" src="image-7.png" width="250" />

The new commit (**Merge Commit**)

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

## Option 1: Rebase

## Nuance

After fix the conflicts:

- `Merge`: You need to make a **new commit** indicating how you solve the conflicts, from which your teammates can straightforwardly see how you did that.

- `Rebase`: You **modify the existent commit**, in this modified commit your teammates may see how you solve the conflicts.

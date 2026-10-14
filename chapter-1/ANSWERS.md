# Answers: SVN Basics in Practice

Model answers to the questions from the [SVN Basics homework](./README.md). Try to answer them yourself first!

## 1. Developer on a plane without internet

> In Step 4 `svn commit` sent your change to the server at once. What does this mean for a developer who works on a plane without internet? How is it different in Git?

**SVN:** the developer can still edit files and run `svn status`, `svn diff` and `svn revert`, because the working copy keeps a clean copy of the checked-out revision. They cannot run `svn commit`, `svn log` or `svn update`, because all of these need the server. A whole flight of work therefore piles up as one big uncommitted change, with no save points in between.

**Git:** the whole repository is on the laptop, so `git commit` and `git log` work offline. The developer makes as many small commits as they like and sends them all with `git push` after landing.

## 2. Staging area

> SVN has no staging area. Name one situation where the Git staging area helps you, and one where it only adds an extra step.

**Where it helps:** you fixed a bug and, in the same session, also left debug output or started an unrelated change. With `git add <file>` you put only the bugfix into the commit and leave the rest for a later one. This is how you keep commits small (atomic), as Chapter 1 recommends.

**Where it only adds a step:** you changed one line in one file and want to save exactly that. In Git you still type `git add` and then `git commit`. In SVN one `svn commit` is enough.

💡 SVN can still commit selectively by naming files, as in `svn commit notes.txt`, but only per whole file.

## 3. Numbers versus SHA1 hashes

> Why can SVN use simple numbers (`r1`, `r2`, `r3`) for revisions while Git needs SHA1 hashes?

**SVN:** there is one central server, and every commit goes through it. The server sees the commits one at a time, so it can simply give each the next number.

**Git:** there is no single place that could hand out numbers. Alice and Bob can both commit offline at the same moment, and both commits would be "number 4". Git needs an identifier that each computer can calculate on its own and that will not clash with anyone else's. A SHA1 hash of the commit's content (files, parent commit, author, date, message) gives that.

💡 The hash also protects integrity. If any part of the history changes, the hash changes.

## 4. Losing the server

> If the folder `/srv/svn/club-repo` is deleted, what is lost? What is lost in Git if the server disappears but every developer still has a clone?

**SVN, `/srv/svn/club-repo` deleted:** the entire history is lost: all revisions, commit messages and authors. Alice and Bob each keep only the files of the one revision they last checked out or updated to. Their working copies cannot commit, update or show the log any more. Without a backup of the server, the project history cannot be recovered.

**Git, server gone:** almost nothing is lost. Every clone holds the full history, so any developer can push it to a new server. What can be lost is only what never reached a clone:

- commits or branches that nobody fetched;
- things that live only on the hosting service, such as pull requests, issues and access settings.

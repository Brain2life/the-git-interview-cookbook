# Homework: SVN Basics in Practice

While reading Chapter 1 we met Git's older alternative, [Apache Subversion](https://subversion.apache.org/) (SVN). In this task you install SVN on an Ubuntu virtual machine and repeat the Git commands from Chapter 1 with their SVN analogues.

The goal is not to learn SVN in depth. It is to feel the difference between a **centralized** version control system (SVN) and a **distributed** one (Git) with your own hands.

**Time:** about 20 minutes

## What you need

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://developer.hashicorp.com/vagrant/install)
- The [Vagrantfile](./Vagrantfile) from this folder

The lab was tested with Ubuntu 22.04 and Subversion 1.14.1.

## Git to SVN cheat sheet

These are the only commands this task covers.

| What you want to do | Git | SVN | Step |
|---|---|---|---|
| Create a repository | `git init` | `svnadmin create <path>` | 2 |
| Get a copy to work in | `git clone <url>` | `svn checkout <url>` | 3 |
| Start tracking a file | `git add <file>` | `svn add <file>` | 4 |
| See what changed | `git status` | `svn status` | 4 |
| Save changes | `git commit` (local only) | `svn commit` (goes straight to the server) | 4 |
| See history | `git log` | `svn log` | 5 |
| Get teammates' changes | `git pull` | `svn update` | 6 |
| Commit identifier | SHA1 hash (`a5afc91`) | Revision number (`r2`) | 7 |
| Discard local changes | `git restore <file>` | `svn revert <file>` | 8 |

## Step 0. Start the virtual machine

On your own computer, in this folder:

```bash
vagrant up
vagrant ssh
```

All commands from here on run **inside the VM**. Your prompt should look like `vagrant@svn-lab:~$`.

## Step 1. Install SVN

```bash
sudo apt update
sudo apt install -y subversion
```

Verify and always check!

```bash
svn --version --quiet
```

```
1.14.1
```

The package gives you several programs. You will use two of them:

- `svn` is the client. You use it every day, like `git`.
- `svnadmin` is the server-side tool. You use it to create repositories.

## Step 2. Create the central repository (`git init`)

In Git every clone is a full repository. In SVN there is **one central repository** on a server, and everybody connects to it. In this lab the "server" is just a folder on the same VM.

```bash
sudo mkdir -p /srv/svn
sudo chown vagrant:vagrant /srv/svn
svnadmin create /srv/svn/club-repo
ls /srv/svn/club-repo
```

```
README.txt  conf  db  format  hooks  locks
```

This folder is the repository database. You never edit files in it by hand, the same way you never edit files inside `.git`.

## Step 3. Check out a working copy (`git clone`)

You will play two developers, Alice and Bob. Start as Alice.

```bash
cd ~
svn checkout file:///srv/svn/club-repo alice
```

```
Checked out revision 0.
```

```bash
cd alice
ls -a
```

```
.  ..  .svn
```

The `.svn` folder is the working copy metadata. Unlike `.git`, it does **not** contain the project history. The history lives only on the server.

```bash
svn info
```

Find the `URL` and `Revision` lines in the output. The repository is empty, so the revision is `0`.

## Step 4. Add, status, commit (`git add`, `git status`, `git commit`)

Create a file and check the status:

```bash
echo "# Reading Club Notes" > notes.txt
svn status
```

```
?       notes.txt
```

`?` means the file is not under version control (Git calls it "untracked").

```bash
svn add notes.txt
svn status
```

```
A       notes.txt
```

`A` means the file is scheduled for addition.

```bash
svn commit --username alice -m "Add reading club notes file"
```

```
Adding         notes.txt
Transmitting file data .done
Committing transaction...
Committed revision 1.
```

💡 `svn commit` sends the change **to the central repository immediately**. It is `git commit` and `git push` in one command. You cannot commit without access to the server.

Now change the file and commit again:

```bash
echo "Chapter 1: Local Git" >> notes.txt
svn status
```

```
M       notes.txt
```

`M` means modified. Commit it straight away:

```bash
svn commit --username alice -m "Add chapter 1 title to notes"
```

```
Sending        notes.txt
Transmitting file data .done
Committing transaction...
Committed revision 2.
```

💡 You did not run `svn add` the second time. SVN has **no staging area**: `svn add` only starts tracking a new file, and after that every modified tracked file goes into the next commit.

## Step 5. Look at history (`git log`)

```bash
svn log
```

```
------------------------------------------------------------------------
```

Empty! Where are your two commits?

They are on the server. Your working copy folder is still recorded as being at revision 0, and `svn log` shows history up to the revision of the folder you are in. Bring the working copy up to date and try again:

```bash
svn update
svn log
```

```
------------------------------------------------------------------------
r2 | alice | 2026-10-12 15:16:33 +0000 (Mon, 12 Oct 2026) | 1 line

Add chapter 1 title to notes
------------------------------------------------------------------------
r1 | alice | 2026-10-12 15:16:33 +0000 (Mon, 12 Oct 2026) | 1 line

Add reading club notes file
------------------------------------------------------------------------
```

## Step 6. Get a teammate's changes (`git pull`)

Now become Bob. Bob gets his own working copy of the same repository and commits a change.

```bash
cd ~
svn checkout file:///srv/svn/club-repo bob
cd bob
echo "Chapter 2: Remote Git" >> notes.txt
svn commit --username bob -m "Add chapter 2 title to notes"
```

```
Committed revision 3.
```

Go back to Alice. Her file does not have Bob's line yet. Ask the server what is new, without changing anything:

```bash
cd ~/alice
svn status -u
```

```
        *        2   notes.txt
Status against revision:      3
```

The `*` means a newer version of the file exists on the server. Get it:

```bash
svn update
```

```
Updating '.':
U    notes.txt
Updated to revision 3.
```

```bash
cat notes.txt
```

```
# Reading Club Notes
Chapter 1: Local Git
Chapter 2: Remote Git
```

💡 Make it a habit: run `svn update` before you start new work, the same way you run `git pull`.

## Step 7. Revision numbers (SHA1 hash)

Git identifies a commit by its SHA1 hash, for example `a5afc91`. SVN uses a simple counter for the whole repository: `r1`, `r2`, `r3`. You already saw them in `svn log` and after every commit.

Use the `-r` option wherever you would give Git a hash.

Show one revision with the files it changed (like `git show <sha1_hash>`):

```bash
svn log -v -r 2
```

```
------------------------------------------------------------------------
r2 | alice | 2026-10-12 15:16:33 +0000 (Mon, 12 Oct 2026) | 1 line
Changed paths:
   M /notes.txt

Add chapter 1 title to notes
------------------------------------------------------------------------
```

Print a file as it was in an old revision:

```bash
svn cat -r 1 notes.txt
```

```
# Reading Club Notes
```

Show the revision of your working copy:

```bash
svn info --show-item revision
```

```
3
```

## Step 8. Discard local changes (`git restore`)

Make a change you do not want to keep:

```bash
echo "oops" >> notes.txt
svn status
```

```
M       notes.txt
```

Throw it away:

```bash
svn revert notes.txt
svn status
```

```
Reverted 'notes.txt'
```

`svn status` prints nothing now, and the file is back to its last committed state.

## Step 9. Clean up

Leave the VM and remove it, on your own computer:

```bash
exit
vagrant destroy -f
```

## Questions

Answer in your own words.

1. In Step 4 `svn commit` sent your change to the server at once. What does this mean for a developer who works on a plane without internet? How is it different in Git?
2. SVN has no staging area. Name one situation where the Git staging area helps you, and one where it only adds an extra step.
3. Why can SVN use simple numbers (`r1`, `r2`, `r3`) for revisions while Git needs SHA1 hashes?
4. If the folder `/srv/svn/club-repo` is deleted, what is lost? What is lost in Git if the server disappears but every developer still has a clone?

## Useful links

- [Version Control with Subversion (the free SVN book)](https://svnbook.red-bean.com/)
- [Apache Subversion quick start](https://subversion.apache.org/quick-start)
- [Workshop: Fundamentals of SVN](https://www.youtube.com/watch?v=E6apGDzqU78)

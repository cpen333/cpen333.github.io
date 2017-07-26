---
layout: lecture
title:  Introduction to Git
date:   2017-07-12 17:50:00
authors: [C. Antonio Sánchez]
categories: [git, revision control]

---

# Introduction to Git
{:.no_toc}

**Authors:** C. Antonio Sánchez (UBC)

{% include toc.html %}

## Learning Goals

In this lecture, we give an introduction to the *version control system* (VCS) called Git.  By the end, you should be able to

* Describe some advantages of version control systems
* Recognize and define key terms related to revision control
* Create a local Git repository from scratch
* Clone an existing repository from a remote location
* Create new branches in your repository and switch between them
* Merge changes between branches
* Add references to remote repositories and push/pull changes between them
* Resolve conflicts that arise when trying to merge changes

It is not *absolutely* necessary to learn how to use Git properly for this course.  However, we will be distributing source code and other materials through Git (including these lectures).  It is a very powerful and useful tool that you will probably encounter in the future (if you haven't already).  I *strongly* suggest you give it a try.

## What is Version Control?

Have you ever found youself making duplicates of a file for the purpose of having a back-up before making changes (e.g. `report_v1.doc`, `report_v2.doc`, `report_final.doc`, `report_final2.doc`, ...)?  Have you ever written code, compiled, ran your program, realized you messed everything up, and wish you could go back to your original?  Version control is designed to help with this.

Version control is a system that lets you save the current "state" of your files so that you can:

- compare changes to files over time
- track versions of an entire project, not just single files
- attach messages to modifications to describe issues or reasons for them
- see who last modified specific files
- go back to previous versions of a file or of the entire project
- etc...

These systems become particularly useful when collaborating with others.  Modern version control systems are designed with collaboration in mind, allowing multiple people to work independently on a set of files, then merge all the changes together automatically.

## Terminology

When working with a version control system, the following terms are often used to describe the "parts" of the system:
<dl>
<dt>Repository</dt>
<dd>The local or remote storage of the versions of the project.</dd>
<dt>Working copy</dt>
<dd>A local, editable copy of the project so we can work on it.</dd>
<dt>File</dt>
<dd>A single file in the project.</dd>
<dt>Version</dt>
<dd>A record of the contents of the project at a point in time.</dd>
<dt>Change</dt>
<dd>The difference between two versions of a single file.</dd>
<dt>Changeset</dt>
<dd>The difference between two complete project versions, usually treated as an indivisible group because of dependencies between the changes.</dd>
<dt>Head</dt>
<dd>The latest saved version.</dd>
</dl>
<br/>

We also have a set of actions for manipulating the versions:
<dl>
<dt>Checkout</dt>
<dd>Create a local working copy from a particular version in the repository.</dd>
<dt>Clone</dt>
<dd>Create a separate copy of the entire repository, with all its history.</dd>
<dt>Commit</dt>
<dd>Save the current state into a new version.</dd>
<dt>Branch</dt>
<dd>Create a division-point in the repository that allows you to diverge from the main content starting at this particular version.  This lets you try things out without affecting the "main" branch (in Git often called the <em>master</em> branch)</dd>
<dt>Merge</dt>
<dd>Pulling together changes in the project from multiple sources to form an updated version.  Changes can be from different users who are sharing their latest modifications, or from different branches to combine functionality (e.g. perhaps you created a separate branch to try something you weren't sure would work, then when it <em>did</em> work, you wanted to integrate it back into the main project).  If a single file is modified by multiple people and the system cannot reconcile the changes, a <em>conflict</em> occurs which must be resolved.</dd>
<dt>Pull</dt>
<dd>Retrieve and merge the lastest changes from some remote repository.</dd>
<dt>Push</dt>
<dd>Send and merge your latest changes into some remote repository.</dd>
<dt>Resolve</dt>
<dd>Addressing a conflict that arises when trying to merge concurrent changes to the same content in a file.  This usually involves deciding between a set of changes, or manually modifying the conflicted portions to merge the multiple changes.</dd>
</dl>

## Why Git?

Git is one of the newer version control systems on the block, having its start around 2005.  In the last few years, its popularity and use has exploded, thanks in part to online cloud storage and hosting providers like [GitHub](https://github.com).  According to [OpenHub](https://www.openhub.net/repositories/compare), Git is now (2017) used by 46% of all open-source projects (or at least of the ones that are tracked by OpenHub, which consist of a list of about 700k).  The next highly-used VCS is *Subversion*, which falls at 41%.  Just a year ago the roles were reversed.  Git is also being adopted by many major commercial companies.  For example, Microsoft is also currently [moving most of its source code over to Git](https://techcrunch.com/2017/05/24/microsoft-now-uses-git-and-gvfs-to-develop-windows/).

Git differs from Subversion and other older version control systems in that it is *distributed*: everybody who works on a project has a separate copy of the entire repository, including all of its history.  In order systems, there was a single centralized repository, and everyone had to connect to it to get the latest changes or save their new versions.  This meant you always needed to be connected to the server if you wanted the benefits of version control.  With a *distributed version control system* (DVCS), you have a complete local copy of the repository, so can always work locally, offline. If you later want to share your changes, others can either connect directly to you to `pull` your updates, or you can `push` your version to some remote repository.

With the popularity of GitHub and other online hosting providers, it has become common practice to treat one version of the repository as the "master copy", usually on some remote server, even though Git was never officially designed for this.  In some sense, modern Git has become a fusion of both the centralized and distributed models.

## Learn Git by Example

For what follows, we are assuming you have Git installed and have access to a unix-style terminal.  If you don't have Git installed, you can download it from [here](https://git-scm.com/download).  On Windows, this installation will come with "Git Bash" which can be used as the unix-style terminal.  On OSX, you can use the [Terminal](http://www.macworld.co.uk/feature/mac-software/how-use-terminal-on-mac-3608274/) app.

We are going to start by creating a small repository from scratch to get you used to the process.  Open the terminal.  It should present you with a prompt that looks something like this:
```
username@machine:folder$
```
with a flashing cursor at the end.  In what follows, I will omit the first part, and just leave the `$` to indicate the prompt at which you type commands.

Your terminal is currently running commands in a certain folder.  You can see the contents of this folder by typing the command `ls` and hitting the *Enter* key.  `ls` stands for "list" on unix systems.  This is similar to the windows `DIR` command some users may be familiar with.  We can see the full path of the directory we are in using `pwd` (present-working-directory).  We can make a *new* directory using the command `mkdir <name_of_new_directory>`, and change our current working directory with `cd <path_to_directory>`.

### Folder Setup

We are going to make a new directory called `cpen333`, and create a new subfolder in there called `hellogit` which will hold our new local repository.  In the terminal, we do this with the following set of commands:
```
$ mkdir cpen333
$ cd cpen333
$ mkdir hellogit
$ cd hellogit
```
We can see our new current working directory using the `pwd` command
```
$ pwd
/c/Users/antonio/cpen333/hellogit
```
Let's add a simple text file using the command-line:
```
$ echo My first git project > readme.txt
```
This creates a new file called `readme.txt`, and sents the content "My first git project" into it.  You can see that the file exists using the `ls` command, and read the contents using `cat <filename>`:
```
$ ls
readme.txt

$ cat readme.txt
My first git project
```

It is very common (outside of the Visual Studio world) to put source files into a folder called `src` and to compile binary files into a separate folder called `bin`.  We are going to create these two folders, and add a couple of text files.
```
$ mkdir src bin
$ cd src
```
The first command created both directories at once.  We should now be in the `src` folder (you can verify this by checking your present working directory).  We are going to create a new C\+\+ source file here.  You can use whatever editor you want.  Here we will use **TextEdit** on OSX or **Notepad** on Windows.

**OSX:**
```
$ touch helloworld.cpp
$ open -a TextEdit helloworld.cpp
```

**Windows:**
```
$ notepad helloworld.cpp
```
Enter the following contents, save the file, and close the editor:
```cpp
#include <iostream>
int main() {
  std::cout << "Hello, world!" << std::endl;
  return 0;
}
```
This is your basic "hello world" program.  We are going to change our working directory back "up" one directory to the `hellogit` folder.  The `..` path is used to denote "back up one directory" (and `.` is the current directory, `..` is up one, `../..` is up two, etc...).
```
$ cd ..
$ pwd
/c/Users/antonio/cpen333/hellogit
$ ls
bin/  readme.txt  src/
```
We should now have  a directory structure that looks like this:
![folder structure]({{site.url}}/assets/lectures/git/git_folder_structure.png)

If you have **gcc** installed, or some other compiler, try compiling the source file into the `bin` folder.  On Windows, I suggest downloading the [MinGW compiler suite](https://sourceforge.net/projects/mingw-w64/), installing it, and adding the `mingw64/bin` to your [Windows `PATH` variable](http://www.mingw.org/wiki/Getting_Started#toc7).  Using **gcc**, we can do the compilation with
```
$ g++ -o bin/helloworld src/helloworld.cpp
```
You can then test that the new program runs successfully
```
$ bin/helloworld
Hello, world!
```
If you don't want to try compiling the program from the terminal, simply create a new file in the `bin` folder to follow along:
```
$ echo Fake binary file > bin/helloworld.exe
```


### Git initial settings

If this is your first time using Git, you will need to initialize some settings.  This is a one-time thing.  Enter your name and email in place of `<...>`:
```
$ git config --global user.name "<your name>"
$ git config --global user.email "<your email>"
$ git config --global color.ui true
```
This sets your name and email address so that when you save revisions, Git knows who you are and can attach your information to your changes.

### Initializing a new repository

To start a new empty Git repository, use
```
$ git init
Initialized empty Git repository in C:/Users/antonio/cpen333/hellogit/.git/
```
Note that Git has now created a folder called `.git` inside your project.  ***NEVER DELETE THIS FOLDER***.  This contains the entire repository and all its version history.  We can check the status of our repository using
```
$ git status
On branch master

Initial commit

Untracked files
  (use "git add <file>..." to include in what will be committed)

        bin/
        readme.txt
        src/

nothing added to commit but untracked files present (use "git add" to track)
```
Git has created a new *master* branch for us, but there is nothing in our repository yet. Git tells us there are "untracked files" (not under version control).  We can add them to our repository using
```
$ git add .
```
This adds all files or modifications in the current directory (`.`).  If we had only wanted to add files in the `src` directory, we could have used `git add src`.  Checking the status of our repo now, 
```
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   bin/helloworld.exe
        new file:   readme.txt
        new file:   src/helloworld.cpp
```
Git now detects that we have "changes" that we wish to save.  

**NOTE:** when we "add" files or changes, git doesn't save them yet to the repository.  Instead, it puts the changes in a *staging area*.  To actually save the changes to the repo, we need to *commit* the added changes.

As a general rule, ***NEVER*** commit your compiled binary files to the repository.  Compiled files are usually system-dependent, the file sizes are often larger than your source files, the entire file changes when you re-compile even if you don't modify any of the source code, and there's no point as long as the source files are committed: whoever checks out your source files can compile the code themselves.  We want to remove our `bin/helloworld.exe` from the set of changes before we save them to the repository.  As Git kindly tells us, we do this with the `git rm --cached` command:
```
$ git rm --cached bin/*
```
which tells us to remove all changes in the staging area from the `bin` folder (the `*` says to match anything in the directory).  If you had omitted the `--cached` part, git would also have deleted the files.  With the `--cached`, it only removed the files from the staging area.

We will now *commit* our changes.  When we do this, we need to provide a meaningful message that tells others (and ourselves) what we did.  This message will appear in the version history along with our changes.
```
$ git commit -m "Initial repo"
[master (root-commit) 127e21d] Initial repo
 2 files changed, 6 insertions(+)
 create mode 100644 readme.txt
 create mode 100644 src/helloworld.cpp
```
This saved our changes to version control, and gives some stats about the files.  We can check the status of our repository with `git status`
```
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

        bin/

nothing added to commit but untracked files present (use "git add" to track)
```

### Ignoring files

Since we ***NEVER*** want to commit our binary files, it is a good idea to tell Git to *always* ignore the `bin` folder.  We can do this by creating a `.gitignore` file and adding it to our repository.
```
$ echo /bin > .gitignore
$ git add .gitignore
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitignore
```
Now our status doesn't mention anything about the `bin` folder, because we told Git to completely ignore it.  Other files you may wish to ignore include Visual Studio project files/solutions (since other users may have their own custom project settings or use different IDEs) and any temporary files.  We can append to the `.gitignore` file using the terminal with `>>`:
```
$ echo *.sln >> .gitignore
$ echo *.vcxproj* >> .gitignore
$ echo Debug >> .gitignore
$ cat .gitignore
/bin
*.sln
*.vcxproj*
/Debug
```
Once we are happy with the set of files being ignored, we can commit our `.gitignore` file
```
$ git add .gitignore
$ git commit -m "added gitignore file"
[master c2753cd] added gitignore file
 1 file changed, 4 insertions(+)
 create mode 100644 .gitignore
```
This added our file and saved the changes to the repository.

### Viewing changes

To view the last few commit log messages, we can use
```
$ git log
Author: Antonio Sanchez <antonio@email.ca>
Date:   Mon Jul 24 14:16:07 2017 -0700

    added gitignore file

commit 127e21d3d6618708dd197705e1ba647c3cfd584a
Author: Antonio Sanchez <antonio@email.ca>
Date:   Mon Jul 24 14:04:54 2017 -0700

    Initial repo
```
Let's change one of the files.  Our `readme.txt` wasn't very specific, so let's change it to
```
$ echo A simple "Hello World" program > readme.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
Git tells us that the file `readme.txt` has been modified, but that those changes are not yet in the staging area to commit.  We can see the difference between our current state and the last one saved in the repository with
```
$ git diff
diff --git a/readme.txt b/readme.txt
index 77adb66..00f001d 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1 +1 @@
-My first git project
+A simple Hello World program
```
This is in *diff* syntax, listing the differences between all our files and the last versions committed.  Here it is telling us that the file `readme.txt` has one line removed (prefixed with `-`) and one line added (refixed with `+`).  Once we are happy with our changes, we can add them to the staging area, then commit those changes with
```
$ git add .
$ git commit -m "better readme"
```

### Git Basics Review

<dl>
<dt><code>git init</code></dt>
<dd>Starts a new repository.</dd>
<dt><code>git add</code></dt>
<dd>Adds files/modifications to the staging area, ready for the next commit.</dd>
<dt><code>git commit</code></dt>
<dd>Saves the added state to the repository.</dd>
<dt><code>git status</code></dt>
<dd>Lists any unstaged files or modifications.</dd>
<dt><code>git log</code></dt>
<dd>Lists information about the last few commits.</dd>
<dt><code>git diff</code></dt>
<dd>Lists changes since the last commit.</dd>
</dl>

## Branches

What do you do if you want to experiment with code before being stuck with it?  You may also want to keep a copy of the existing code-base in case things fail miserably.  The old-school approach is to make a backup copy of your entire project folder.  With branching, you don't need to do this.  Instead, you can start a new branch, work along in the branch, adding changes/committing versions.  You can easily switch between branches at any point, going to the main (i.e. *master*) branch to see your backed-up code, and back to your *experimental* branch to see your new stuff.

![git branches]({{site.url}}/assets/lectures/git/git_branch.png)

*Three branches in Git, the experimental branch diverged at some point, but then once the code was tested and ready, its changes were merged back into the master branch.*

New branches start at the current version, then can continue independently, with their own set of commits.  We are going to start a new branch from here in our little test project called *experimental*:
```
$ git branch experimental
$ git branch -l
  experimental
* master
```
The `-l` switch lists the set of existing branches, with the `*` indicating that we are still currently on the *master* branch.  We can switch to the experimental branch using the *checkout* command
```
$ git checkout experimental
Switched to branch 'experimental'
$ git branch -l
* experimental
  master
$ git status
On branch experimental
nothing to commit, working tree clean
```
When listing branches, git now tells us we are on `experimental`, and the Git status message also informs us we are on the *experimental* branch.  Let's modify our code for `helloworld.cpp` and commit to this new branch.

**helloworld.cpp:**
```cpp
#include <iostream>
int main(int argc, char *argv[]) {
  std::cout << "Hello, world!" << std::endl;
  
  std::cout << "Arguments:" << std::endl;
  for (int i=0; i<argc; ++i) {
    std::cout << "  " << argv[i] << std::endl;
  }
  return 0;
}
```

```
$ git add src/helloworld.cpp
$ git commit -m "Print out arguments"
[experimental 629aa4d] Print out arguments
 1 file changed, 16 insertions(+), 2 deletions(-)
```
These changes were only saved to the *experimental* branch.  If we switch back to the *master* branch, we should see our original code.
```
$ git checkout master
Switched to branch 'master'
$ cat src/helloworld.cpp
#include <iostream>
int main() {
  std::cout << "Hello, world!" << std::endl;
  return 0;
}
```
Let's say we're happy with our modifications over in the *experimental* branch, and we want to bring them into the main code.  We use the *merge* command for this:
```
$ git merge experimental
Updating 2c2831f..629aa4d
Fast-forward
 src/helloworld.cpp | 18 ++++++++++++++++--
 1 file changed, 16 insertions(+), 2 deletions(-)

$ cat src/helloworld.cpp
#include <iostream>
int main(int argc, char *argv[]) {
  std::cout << "Hello, world!" << std::endl;
  
  std::cout << "Arguments:" << std::endl;
  for (int i=0; i<argc; ++i) {
    std::cout << "  " << argv[i] << std::endl;
  }
  return 0;
}
```
As we can see, this brought over all the changes from the other branch, *experimental*, into our current branch, *master*.  Since nobody had modified the exact same lines in *master* in the meantime, Git was able to complete the merge successfully.

### Git Branching Review

<dl>
<dt><code>git branch</code></dt>
<dd>Creates a new branch from the current version.</dd>
<dt><code>git checkout</code></dt>
<dd>Switches to a different branch.</dd>
<dt><code>git merge</code></dt>
<dd>Merges contents from another branch into the current one.</dd>
</dl>

## Remote Repositories

For what follows, we are going to be working with the course's remote repositiory on **GitHub**, and you will need your own remote repository to save your solution.  You may wish to use *private* repositories for your lab/project work to prevent others from finding and copying your solutions.

If you already have a GitHub account, great :).  Students should be able to register with a *student* account, and get a few free private repositories.  If you don't have any more free private repositories available, or if **GitHub** doesn't recognize your student status, I recommend setting up an account at [GitLab.com](gitlab.com).  They promise to always allow an unlimited number of private repositories.

You can think of **GitHub** or **GitLab** as a cloud-storage for the master copy of your repository.  By keeping it up-to-date, you will be able to access your project any time, from anywhere, on any computer, as long as you have internet access.  For this exercise, you will be reading an initial repository for the [CPEN 333 GitHub account](github.com/cpen333).  Since you don't have write-access to this account, you will need to save your work to your own repository hosted somewhere.

![git remote submission]({{site.url}}/assets/lectures/git/git_remote_submission.png)

*We will* `clone` *a copy of the remote repository from GitHub to create a local copy, modify it, then push our modified version to a new cloud-based storage location*

To start, let's *clone* the **cubicsolver** repository from **GitHub** to your local machine.  If you are still in the **hellogit** repository from before, change up to its parent directory using `cd ..`.  To clone a repository, use
```
$ git clone https://github.com/cpen333/cubicsolver.git
Cloning into 'cubicsolver'...
remote: Counting objects: 26, done.
remote: Compressing objects: 100% (20/20), done.
remote: Total 26 (delta 7), reused 24 (delta 5), pack-reused 0
Unpacking objects: 100% (26/26), done.

$ cd cubicsolver
```
This will have created a new folder called `cubicsolver` on your local machine that contains a complete copy of the repository.  We changed into this directory.  Git remembers where we obtained our cloned repository from.  You can see this by checking
```
$ git remote -v
origin  https://github.com/cpen333/cubicsolver.git (fetch)
origin  https://github.com/cpen333/cubicsolver.git (push)
```
Git has set up a "remote" storage location which it named `origin` that points to the original GitHub repository.  The *(fetch)* location indicates that we call *pull* new changes from `origin` if someone else ever updates it.  The *(push)* location indicates that we can *push* our changes back up to `origin` (though if you tried, you would get an error since you don't have write permission on the server).

We are now going to set up a different remote on either GitHub or GitLab.  Sign in to your account and create a new repository online.  On GitHub, you do this by clicking on the `+` icon in the top-right next to your avatar and selecting **New repository**.  In GitLab, you click on the `+` icon on the top-right toolbar and select **New project**.  Give the new repository the name **cubicsolver**, give it a short description, then click on **Create repository**/**Create project**.  GitHub/GitLab will then give you some brief instructions on how to commit your initial code.  We will modify these instructions slightly to account for the fact that we will be working with multiple remotes.

Back in the terminal, add a new remote repository
```
$ git remote add submission https://<remote_url>/cubicsolver.git
```
Replace `<remote_url>` with your GitHub or GitLab account url (e.g. `gitlab.com/antonios`).  We have just created a *remote* location that we can push or pull changes to/from.  We named this remote *submission*, but you could have used any other name.  If we now try
```
$ git remote -v
origin  https://github.com/cpen333/cubicsolver.git (fetch)
origin  https://github.com/cpen333/cubicsolver.git (push)
submission      https://gitlab.com/antonios/cubicsolver.git (fetch)
submission      https://gitlab.com/antonios/cubicsolver.git (push)
```
we see that we have two different remote locations.  We can get changes from or push changes to either of them (assuming we have the correct permissions to do so).  Let's push our current local repository up to `submission`:
```
$ git push submission master
Counting objects: 16, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (16/16), 4.51 KiB | 0 bytes/s, done.
Total 16 (delta 3), reused 0 (delta 0)
To https://gitlab.com/antonios/cubicsolver.git
 * [new branch]      master -> master
```
This  *pushed* the contents of your `master` branch up to the remote repository you named `submission`.  It contains all the contents and history of our current *master* branch.  You can similarly push any other branches you want up to your new remote repository.  Go back to your GitHub or GitLab account and verify that you see all your files there.

Next, we will *pull* new changes from the remote repository to our local one.  If you try
```
$ git pull submission master
From https://gitlab.com/cpen333/cubicsolver
 * branch            master     -> FETCH_HEAD
Already up-to-date.
```
It should tell you that you are already up-to-date.  Let's say you're working with a partner, and he or she has just committed some code.  We will emulate this by signing into either GitHub or GitLab, and modifying the source file `src/main.cpp` through the web interface.  If you navigate to and click on the file, you should see the option to `Edit` it.  Change some of the contents, then select `Commit` at the bottom of the page.  Now, on your local machine, you should be able to quickly get and merge those new changes with
```
$ git pull submission master
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 2), reused 0 (delta 0)
Unpacking objects: 100% (4/4), done.
From https://gitlab.com/cpen333/cubicsolver
 * branch            master     -> FETCH_HEAD
   5bf123c..56f4b62  master     -> submission/master
Updating 5bf123c..56f4b62
Fast-forward
 src/main.cpp | 2 ++
 1 file changed, 2 insertions(+)
```
If you look at the contents of your local files, they should now match the updated ones on the remote server.  Here we pulled changes from our own `submission` remote, but we could pull changes from `origin`, or any other repository that shares some project history.

### Git Remote Review

<dl>
<dt><code>git clone</code></dt>
<dd>Creates a local copy of a remote repository.</dd>
<dt><code>git remote add</code></dt>
<dd>Adds a new remote repository reference location.</dd>
<dt><code>git pull</code></dt>
<dd>Pull the latest updates from a remote repository.</dd>
<dt><code>git push</code></dt>
<dd>Pushes your latest updates to a remote repository.</dd>
</dl>

## Resolving Conflicts

So far, we (hopefully) haven't run into any conflicts when pushing or pulling changes.  This is because we only ever modified one file on one machine at a time.  If you are working with collaborators, however, you'll often run into cases where multiple people try to modify the same section of the same file.  If you then try to either push or pull changes to that file, BAM! conflict.

To set up a conflict, choose to modify a particular line in one of the source files.  For instance, change the `src/main.cpp` file to remove the comment on line 10:
```cpp
  // YOUR CODE HERE //
```
and replace it with some code.  Add the changes and commit to your local repo, but DO NOT PUSH them yet.  Next, go to your GitHub/GitLab repository through the web interface, remove the same comment, and replace it with *different* code (if it matches what you have in your local repository, then there won't be a conflict).  Hit the **Commit** button to save those changes.  Back in your terminal, try to `push` your latest update to the server
```
$ git push submission master
To https://gitlab.com/cpen333/cubicsolver.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://gitlab.com/antonios/cubicsolver.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
Git recognizes that you will be over-writing changes that you don't have yet on your local machine, so prevents you from doing this.  It forces you to first pull those changes and deal with any conflicts locally.
```
$ git pull submission master
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 2), reused 0 (delta 0)
Unpacking objects: 100% (4/4), done.
From https://gitlab.com/cpen333/cubicsolver
 * branch            master     -> FETCH_HEAD
   56f4b62..e58556d  master     -> submission/master
Auto-merging src/main.cpp
CONFLICT (content): Merge conflict in src/main.cpp
Automatic merge failed; fix conflicts and then commit the result.
```
Git has tried to pull all the changes from the remote repository, but has detected a *CONFLICT*.  If you open up the file `src/main.cpp`, you will see something like this
```cpp
...
// Solves for all unique roots of f(x) = a*x^3 + b*x^2 + c*x + d
//   populates out[] with the solution
//   returns # roots found
int solve_cubic(double d, double c, double b, double a, double out[]) {
<<<<<<< HEAD
  
  for(int i=0; i<10; ++i) {
    std::cout << "hello" << std::endl;
=======

  for (int j=0; j<20; ++j) {
    std::cout << "goodbye" << std::endl;
>>>>>>> e58556d7cbe8f6b1f20f8722e1a05f06ec3fd4fc
  }

  return 0;
}
...
```
From the `<<<<<<< HEAD` to the `========` markers, this is the code that you have locally at your `HEAD` (i.e. most recent) revision.  From the `=======` to the `>>>>>>> longhexadecimalnumber` marker is the set of changes on the remote repository.  Git will try to merge what it can, but will sometimes run into cases like here where it doesn't know which modifications in this section of code to keep.  It is your job to remove any unwanted code and markers, and add/commit your changes to your local repo.  For example, I might want to keep my local edits, so I would remote the `<<<<<<< HEAD` marker and everything from the `=======` to the `>>>>>>> ...`.  After making the desired edits, you can run
```
$ git add .
$ git commit -m "merged changes from remote"
$ git push submission master
Counting objects: 8, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (8/8), done.
Writing objects: 100% (8/8), 745 bytes | 0 bytes/s, done.
Total 8 (delta 4), reused 0 (delta 0)
To https://gitlab.com/cpen333/cubicsolver.git
   e58556d..7e754cd  master -> master
```
The *push* should now succeed, since you've updated your local copy to the latest and fixed any conflicts.

### Mergetools

When you start running into a lot of conflicts, it can get rather tedious to find and fix them all manually.  There are many tools out there to help you.  My favourite so far on Windows/Linux is [Meld](http://meldmerge.org/).  Once you have **Meld** installed, you can configure Git to use it as your default *merge-tool*:
```
$ git config --global merge.tool meld
$ git config --global mergetool.keepBackup false
$ git config --global mergetool.meld.path "C:\Progra~2\Meld\Meld.exe"
```
The last line is only needed on Windows and should specify the full path to the Meld executable.  To resolve conflicts, you can enter
```
$ git mergetool
```
Meld should then present a graphical interface which will allow you to select which changes you want to keep, and which to ignore.

![meld conflict resolution]({{site.url}}/assets/lectures/git/meld.png)

*Meld presents the conflict and tools for quickly resolving it.  The left pane shows your current local copy, the right the conflicting remote copy, and the center is your working copy where you resolve the conflict.*

It may be possible to get Meld running on OSX.  I have seem some tutorials kicking around, and there is [this site](https://yousseb.github.io/meld/) that offers a custom build of Meld just for OSX and git integration.  It is not an official release from Meld, so buyer beware.

## Challenge

Try solving the cubic-root problem from this lecture.  Feel free to create another branch called `experimental` to try your approach without affecting the *master* branch.  Once you have found a solution, merge it back into `master`.

## Acknowledgements

Some of this material comes from a tutorial on Git for CPEN 221 - Principles of Software Construction, written by Sathish Gopalakrishnan (UBC) and myself (Antonio Sánchez).

## Additional Resources

We've covered quite a bit of material, but there is a lot more Git can do, and you'll eventually run into more complex issues that we haven't touched on.  Luckily, since Git is so popular, you should be able to find solutions to any problem that come up on [StackOverflow](https://stackoverflow.com/documentation/git/topics).  For more practice with Git, try going through the following online tutorials

- Try-Git [(https://try.github.io)](https://try.github.io)
- Atlassian's Git Tutorial [(https://www.atlassian.com/git/tutorials/what-is-git)](https://www.atlassian.com/git/tutorials/what-is-git)
- Git-SCM's Git Tutorial [(https://git-scm.com/book/en/v2/Getting-Started-Git-Basics)](https://git-scm.com/book/en/v2/Getting-Started-Git-Basics)
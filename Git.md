# Git

> ## Other Tips:
>
> ```shell
> git log
> git branch
> git tag
> 
> git checkout <branch> 
> git checkout <tag> 
> git checkout <commit>
> ```
>
> 

In a local Git repository, a file can have one of three states:

- Untracked: a file you create in a repository, but not yet added to Git.
- Tracked: a file that has been added to Git.
- Staged: a *tracked* file that has been changed and added to Git's commit queue.



As long as you clone your GitHub project via SSH, you'll be able to write back to your repository.



```shell
git init .

git add <files>

git rm --cached <files>
git reset HEAD <files>

git commit -m '<some comments.>'

git status

git log [--online]

```



To move HEAD around in your own Git timeline, use the `git checkout` command.

There are two ways to use the `git checkout` command:

1. A common use is to restore a file from a previous commit, and 
2. you can also rewind your entire tape reel and go in an entirely different direction (*Branches*).

> When you rewind the tape in this way, if you hit the record button and go forward, you are destroying your future work. **By default**, Git assumes you do not want to do this, so it detaches HEAD from the project and lets you work as needed without accidentally recording over something you have recorded later.

```shell
git checkout HEAD foo			# Restore a file
git checkout 0e9a20a foo		# 

git checkout b5e1a67			# Rewind the entire Git project (branches)

# to create a new branch
git checkout -b remix			# To get your Git HEAD back down on blank tape, make a new branch
git checkout -b crazy_idea

# change back to your master branch 
git checkout master 			# Forget your ideas in shame and pretend your new branch doesn't exist

# To keep your crazy ideas and pull them back into your master branch
git checkout master				# change back to your master branch and 
git merge crazy_idea			# merge your new branch
```

> **Branches** are powerful aspects of git, and it's common for developers to create a new branch immediately after cloning a repository; that way, all of their work is contained on their own branch, which they can submit for merging to the master branch. 



## Working with remotes

- *local* repository
- *remote* repository

The only **difference** between working on your own private Git repository and working on something you want to share with others is that at some point, you need to `push` your changes to someone else's repository. 

> When you clone a repository with read and write permissions from another source, your clone inherits the remote from whence it came as its *origin*.

Having a remote origin is handy because it is functionally an offsite backup, and it also allows someone else to be working on the project.

```shell
git remote --verbose
	# origin	https://gitlab.com/trashy/trashy.git (fetch)
	# origin	https://gitlab.com/trashy/trashy.git (push)
```

If your clone didn't inherit a remote origin, or if you choose to add one later, use the `git remote` command:

```bash
git remote add origin user@example.com:~/myproject.Git	
```

If you have changed files and want to send them to your remote `origin`, and have read and write permissions to the repository, use `git push`. 

> The first time you push changes, you must also send your branch information. It is a good practice to not work on master, unless you've been told to do so. After you've pushed your branch once, you can drop the `-u` option.

```bash
git checkout -b seth-dev
git add exciting-new-file.txt
git commit -m 'first push to remote'

git push -u origin HEAD			# Have been pushed once, you can drop the -u option.
```



## Merging branches

When you're working alone in a Git repository you can merge test branches into your master branch whenever you want. 

- review their changes before 
- merging them into your master branch

```bash
git checkout remix
cat foo

git checkout master
git merge remix			
```

> If you are using GitHub or GitLab or something similar, the process is different. There, it is traditional to fork the project and treat it as though it is your own repository. You can work in the repository and send changes to your GitHub or GitLab account without getting permission from anyone, because it's *your* repository.
>
> If you want the person you forked it from to receive your changes, you create a *pull request*, which uses the web service's backend to send patches to the real owner, and allows them to review and pull in your changes.
>
> Forking a project is usually done on the web service, but the Git commands to manage your copy of the project are the same, even the `push` process. Then it's back to the web service to open a pull request, and the job is done.



## Graphical tools for Git

- Git in KDE Dolphin
- [Sparkleshare ](http://www.sparkleshare.org/)(synchronization)
- Git-cola (monitor changes)

## Build your own Git server

### Shared Git server

> If you know how to use Git and SSH, then you already know how to create a Git server. The way Git is designed, the moment you create or clone a repository, you have already set up half the server. 
>
> Then enable SSH access to the repository, and anyone with access can use your repo as the basis for a new clone.

- Enable SSH logins using only SSH key authorization. 

- Create the *gituser*, who is the shared user for all of your authorized users

  ```shell
  su -c 'adduser gituser'
  ```

- Switch over to that user, and create an *`~/.ssh`* framework with the appropriate permissions. (Important!) 

  ```shell
  su - gituser
  mkdir .ssh && chmod 700 .ssh
  touch .ssh/authorized_keys
  chmod 600 .ssh/authorized_keys
  ```

- Copy the public keys into the gituser's *authorized_keys* file. 

- Add *git-shell* to your system, and then make it the default shell for your gituser.

  ```shell
  # Run these commands as root:
  grep git-shell /etc/shells || su -c "echo `which git-shell` >> /etc/shells"
  su -c 'usermod -s /usr/bin/git-shell gituser'
  ```

- Add yourself to the corresponding group for the gituser.

  ```shell
  usermod -aG gituser shawn
  ```

- Make a Git repository. 

  ```shell
  git init --bare /opt/jupiter.git
  chown -R gituser:gituser /opt/jupiter.git
  chmod -R 770 /opt/jupiter.git
  
  # any user who is either authenticated as gituser or is in the gituser group 
  # can read from and write to the jupiter.git repository.
  git clone gituser@localhost:/opt/jupiter.git jupiter.clone       
  	# Cloning into 'jupiter.clone'...
  	# warning: You appear to have cloned an empty repository.
  ```

> Make it a bare repository, which has no *working tree* (that is, no branch is ever in a `checkout` state). This is important because remote users are not permitted to push to an active branch. Since a bare repo can have no active branch.
>
> Place this repository anywhere you please, just as long as the users and groups you want to grant permission to access it can do so, for instance in a common shared location, such as */opt* or */usr/local/share.*

**Remember**: developers MUST have their public SSH key entered into the *authorized_keys* file of gituser, or if they have accounts on the server (as you would), then they must be members of the gituser group.

## Git hooks

[reading](https://opensource.com/life/16/8/how-construct-your-own-git-server-part-6)

## Manage binary blobs with Git

[! read](https://opensource.com/life/16/8/how-manage-binary-blobs-git-part-7)

Git can't do much with an impervious binary file except treat it as one big solid black box and commit it as-is.

[Git Large File Storage](https://git-lfs.github.com/) (LFS) is an open source project from GitHub that began life as a fork of git-media. Git-media and [git-annex](https://git-annex.branchable.com/walkthrough/) are extensions to Git meant to manage large files. They are two different approaches to the same problem, and they each have advantages. These aren't official statements from the projects themselves, but in my experience, the unique aspects of each are:

- `Git-annex` favors a distributed model; you and your users create repositories, and each repository gets a local `.git/annex` directory where big files are stored. The annexes are synchronized regularly so that all assets are available to all users as needed. Unless configured otherwise with `annex-cost`, `git-annex` prefers local storage before off-site storage.
- `Git-portal` is also distributed, and like Git-annex has the option to synchronize to a central location. It uses common utilities that you likely already have installed (specifally Bash, Git, rsync).
- `Git-LFS` is a centralised model, a repository for common assets. You tell Git-LFS where your large files are stored, whether that's a hard drive, a server, or a cloud storage service, and each user on your project treats that location as the central master location for large assets.

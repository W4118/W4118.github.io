# Getting started with Git

Git is a version control system. If you don't know git, you probably want to start by reading the [Pro Git Book](https://git-scm.com/book/en/v2).

It might be helpful to configure the Git settings on the machine you're using for development (in this case, your VM). Create or edit the file `~/.gitconfig` and add the following content:

```
[user]
        name = YOUR_NAME_HERE
        email = YOUR_EMAIL_HERE
[color]
        diff = auto
        status = auto
        branch = auto
[push]
        default = matching
```

While there are a number of different Git-based systems, you will be using Github Classroom for this course. Make sure you have a Github account. If you are new to Github, please check out the [getting started guide](https://help.github.com/articles/set-up-git/).

**Please fill in [this Google Form](https://forms.gle/xxeEgMAN4me55J6A8)** to let us know your Github username and your UNI. This must be done by the last day of the Change of Program period.

For each homework assignment, your team's source code will be stored at `github.com/W4118/f25-hmwkN-UserName` as a private repository. Only you or your group members will be allowed to push/pull from the repository. A repository containing skeleton code for that assignment will also be available at `github.com/W4118/f25-template-hmwkN`. The initial state of your repository will be set to match the skeleton code. Instructions will be provided with each assignment on how to access your repository for the respective assignment. You may visit [our GitHub organization](https://github.com/w4118) to see all the repositories available to you.

Once you have access to your assignment repository, you will need to clone your homework repository locally to start working on it:

```
$ git clone git@github.com:W4118/f25-hmwkN-UserName.git
Initialized empty Git repository in f25-hmwkN-UserName/.git/
Receiving objects: 100% (x/x), done.
remote: Counting objects: x, done.
remote: Total x (delta 0), reused 0 (delta 0)
```

Your typical workflow:

```
$ git diff              # see what you've changed
$ git add file1 file2
$ git diff --cached     # see what you are about to commit
$ git commit            # explain what you did on file1 file2
```

A few commits later:

```
$ git push              # upload your changes to the repository
Counting objects: x, done.
Delta compression using up to x threads.
Compressing objects: 100% (x/x), done.
Writing objects: 100% (x/x), x bytes, done.
Total x (delta 0), reused 0 (delta 0)
To git+ssh://xxx
   xxxxxxx..xxxxxxx  main -> main
```

If git asks you for a password whenever you try to do a clone/push/pull/fetch then you need to make sure that you set up SSH authentication. See the section on `Pushing from your VM to GitHub` in [this guide](./ssh.md) for more details.

**Checking your submissions**

Once you have submitted your homework, we strongly recommend that you re-clone the submission to your machine to check that what we have received is in fact what you intended to submit.

> **Note:** You can always submit again until the deadline. After the deadline, you will not be able to push to the git repository.

The procedure for doing so would be the following:

```
$ pushd /tmp
$ git clone git@github.com:W4118/f25-hmwkN-UserName.git
$ cd f25-hmwkN-UserName

# Check contents of the directory and test aginst your test cases

$ cd ..
$ rm -rf f25-hmwkN-UserName
$ popd
```

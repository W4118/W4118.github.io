#Getting started with Git
##W4118 Fall 2016

Git is a version control system that Linux hackers use. If you don't know git, you probably want to start by reading the [**Pro Git Book**](https://git-scm.com/book/en/v2).

Remember to config your git settings on your developing machine, either physical or virtual, properly. Create (or edit) the file ~/.gitconfig with the following content
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

While there are a number of different Git-based systems, you will be using Github Classroom for this course. Make sure you have a registered Github account. If you are new to Github, please check out the [**getting started guide**](https://help.github.com/articles/set-up-git/). There is also [**ssh setup guide**](https://help.github.com/articles/generating-an-ssh-key/) for you to enable ssh access to Github.

##Caution: If you use the VM image provided on [the homework page](http://www.cs.columbia.edu/~nieh/teaching/w4118/homeworks/), please remove the our testing ssh public/private key pair, and generate or copy your own to the VM. Please do NOT use the testing public key as your Github ssh key.

Also please fill in the [**Google Form**](https://docs.google.com/a/columbia.edu/forms/d/1x_WFznM2y7ts_q6OpgXBOFbXd61Hnzv9NtwCgl6Pgjc) (**Please Login to LionMail first**) to declare your Github user name.

Your source code will be stored at **_github.com/W4118/hmwkN-UserName_** as a private repository. Only you or your group members are allowed to push/pull from the repository. The sample code repository is at **_github.com/W4118/template-hmwkN_**. Your repository will be cloned from the sample repository initially, and you will be working on that repository afterwards. Be carefull not to overwrite those initial commits made by professors or TAs in your repository. Otherwise it would be much harder for TAs to grade your homework.

Each assignment will be published as a Github Classroom invitation link. Click "Accept Assignment" to setup your own copy of the assignment. Please wait until Github finishes the initialization process. You may visit [**W4118 Organization**](https://github.com/w4118) to see all the sample codes and your repositories.

Click on the link below for assignments

###[**Homework 1**](https://classroom.github.com/assignment-invitations/4fc3c84ba63f479d9f39281364b97017)

Then, you will need to clone your homework repository locally to start working on it: 
```
$ git clone git@github.com:W4118/hmwkN-UserName.git
Initialized empty Git repository in hmwkN-UserName/.git/
Receiving objects: 100% (x/x), done.
remote: Counting objects: x, done.
remote: Total x (delta 0), reused 0 (delta 0)
```

Your typical workflow:
```
$ git diff # see what you've changed
$ git add file1 file2
$ git diff --cached # see what you are about to commit
$ git commit # explain what you did on file1 file2
```
A few commits later
```
$ git push # upload your changes on the repository
Counting objects: x, done.
Delta compression using up to x threads.
Compressing objects: 100% (x/x), done.
Writing objects: 100% (x/x), x bytes, done.
Total x (delta 0), reused 0 (delta 0)
To git+ssh://xxx
   xxxxxxx..xxxxxxx  master -> master
```
If git asks you for a password whenever you try to do a clone/push/pull/fetch then you need to make sure that your ~/.ssh/ directory looks like this: 
```
$ ls -lh ~/.ssh
-rw------- 1 root root 672 Aug 17 2007 id_rsa
-rw-r--r-- 1 root root 604 Aug 17 2007 id_rsa.pub
```
If you already use ssh keys for logging into other servers or git repositories, you can save the private/public keypair that was created for you as ~/.ssh/id_rsa and ~/.ssh/id_rsa.pub and create the file ~/.ssh/config like this:
```
$ touch ~/.ssh/config
$ chmod 600
```
Then open the file and input this data:
```
Host github.com
	IdentityFile ~/.ssh/id_rsa
	User git
```

Checking your submissions

Once you have submitted your homework we strongly recommend that your re-clone the submission to your machine to check that what we have received is in fact what you intended to submit.

The procedure for doing so would be the following:
```
$ pushd /tmp
$ git clone git@github.com:W4118/hmwkN-UserName.git
$ cd hmwkN-UserName
# ... check contents of directory and test aginst your test cases
$ cd ..
$ rm -rf hmwkN-UserName
$ popd
```

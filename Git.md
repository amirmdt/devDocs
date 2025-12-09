# Git
***to control version***

*first of all you should install git on your system. For example on rocky linux use : `sudo dnf install git`*
```
cd /tmp
mkdir project
cd project
git init
```
*the above commands will create .git directory in `/tmp/project` directory which let git control the directory `/tmp/project`*

`git status` *this command will show the status of this dir right now*

*now you can create new file like `index.html` and add some text in it*\
*now if you use `git status` it will show you that there is a file named `index.html` which is not tracked by git*\
*you should use `git add index.html` command to send the file to stage mode.*\
*Now the file is ready to be commited by commad below:*
```
git commit -m ‘first file added’
```
__________________
*for the first time you are using `commit` command maybe you get error about email and username:*\
*for that you should add your `username` and `email` to your git config using commands below:*
```
git config --global user.email "you@example.com"
git config --global user.name "cameronmcnz"
```
__________________
*-m option in git commit command is to add a message for this commit*

*now if you add another file `(page1.html)` to this dir and you change the `index.html` file and use `git status` command you will see that there is a file which has changes not staged for commit `(index.html)` and there is untracked file `(page1.html)`*

*you can add all files together using `git add -A` or you can add one by one and commit added files separately.*

*Like below :*
```
git add -A
git commit -m “added page1 and edited index”
(git status)
```
*or* 
```
git add index.html
git commit -m “changed index”
(git status)
git add “page*”
git commit -m “added page1”
(git status)
```
*you can see all commits till now using command below :*\
`git log`

`-git diff`:
*if you change one or more files , you can use `git diff HEAD` command to see changes in files of this file with while they were on last commit*

*if you use git add commad to add changes to stage mode then you can use `git diff --staged` commad to see changes of files which are added to stage mode by the last commit.*

`-git reset`:
*if you want to return one file in stage mode to when it wasnt added ( for example `page3.html` ) you can use below command:*
`git reset page3.html`

*now if you use `git status` command u will see `page3.html` is unstaged again*

`-git checkout`
*if you want to change the unstaged file like `page3.html` to what it was like in last commit (`HEAD`) you can use below command :*\
`git checkout -- page3.html  `

*now if you use git status the file `page3.html` is not in unstaged mode anymore. And if you use cat `page3.html` you will see your changes are gone and the file is like it was in last commit.*

### Branches :

*branches are good when someone want to make changes to `master` simultaneously*\
*you make branches on `master` branch and let each person work on one or if you want to make changes on project with different subjects for example `fixbugs` and `change UI` etc*

*to do that first :*

*you can see all branches using command below:*\
`git branch`

*when you didn’t create any new branch you have only `master` branch*

*to create new branch ( for example branch named `fixpages` ) :*\
`git branch fixpages`

*now if you use `git branch` command you can see new branch named `fixpages` added but you are still on `master` branch.*\
*If you want to go on `fixpages` branch to work there you can use command :*\
`git checkout fixpages`

_________
***you could create and checkout to a branch with a single command like below:***
```
git checkout -b fixpages
```
_________

*now with command `git branch` you can see you are in `fixtpages` branch*

*now if you make change to files or create new files nothing will change on master branch.*\
*For example you make changes and create new files and add and commit them. Then using `git status` see everything is ok and using ls and cat command you can see your changes.*\
*Now if you use `git log` you can see last commits here and on `master`.*\
*Then using `git checkout master` go back to `master` branch. Now you see files which you changed are not changed and your new files are not here anymore. Even if you use `git log` command you cant see commits on `fixpages` branch. If you want to make your changes on `fixpages` branch on `master` branch you should use command below :*\
`git merge fixpages`

*now new files and changes on `fixpages` will be on `master` branch too. And using `git log` command you can see the commits which made on `fixpages` branch.*

***Tip*** *: If you make changes on a branch ( for example on `fixpages` here ) you can see changes on `master` branch until you didn’t commit your changes on the branch ( `fixpages` ) but after commiting you cant see changes on `master` but when you merge the branch changes on the `master` all changes are in `master` too now.*

`-git branch -d branchname`\
*If you are done with a branch ( for example `fixpages` branch ) you can delete that branch using command below :*\
`git branch -d fixpages`

`-git rm filename`\
*using this command you can remove a file from git and filesystem but you have to commit that*

_________________________________________________

## github and gitlab:

*these are remote servers for saving your project.*\
*In a repository which is public or a repository which you have access you can `clone` a project to your local system using by copying the web URL of that repository and using command below :*
`git clone URL`

*when you used that a new directory will be created in the path you are with the name of repository and all files related to that repo will be there. And you can change files and add and commit that changes to your `master` branch ( `master` branch is the one in your `local system` ) then if you have the permission you can push your commits to the `origin` branch ( `origin` branch is the one in `remote [ github ] system` ) using command below :*\
`git push origin master`

*and if someone else changed the `origin` branch you can `pull` last changes from `origin` to `master` using command below :*\
`git pull origin master`\
*it is better to `pull` last changes anytime you want to work on `master` branch*

*you can have a `master` branch on your `local system` and after that you can add a `remote` to this `master` as `origin` using command below :*\
`git remote add origin URL`\
*which `URL` is the `web url` of the `remote repository` that you want to add to this `master` as `origin`
and after this you can use `git remote` or `git remote -v` to see the `origin` and its `url`*\
*and after this you can push your project to that repository using : `git push origin master`*\
*but when I tried to push I got error which said `after 2021 we cant push from local to github using password Authentication`*\
*for that I used `ssh-key`*\
*for that first we should generate a ***public and private ssh key*** using command below :*\
`ssh-keygen -t rsa`\
*then copy contents of pub one and paste it in path below in `github` :*\
***setting > SSH and GPG keys > New SSH key***\
*then I went to repository in my github and copied the SSH path from <> Code
and after that using command below I changed the origin remote path :*\
`git remote set-url origin SSH`\
*which `SSH` is the path I copied above*

*now using command `git push origin master` I could push my project to my repository.*

### Conflicts :
*think you pull the `origin` to `master`. Then make some changes and before you push your commits to `origin` someone else made some changes to `origin` by pushing from another `local`. Then you will have error which is refer to ***conflicts***. Then you have to pull again from `origin` and resolve the conflicts and push again.
First time you want to pull in this situation you will get error which*\ 
*This error occurs because your local `master` branch and the remote `origin/master` branch have diverged (i.e., they have different commit histories). Git doesn’t know how to merge them automatically, so it asks you to choose a strategy.*

#### How to Fix This
*You have 3 options (choose one based on your needs):*

***Option 1:*** *Merge (Keep Both Changes)*\
*This creates a new merge commit combining both branches.*
* bash
* Copy
```
git config pull.rebase false  # Set merge as the default strategy (optional)
git pull origin master        # Pull and merge remote changes
```   
    • Use this if: You want to preserve all commit histories from both branches.

***Option 2:*** *Rebase (Rewrite Local Changes on Top of Remote)*\
*This moves your local commits to the top of the remote branch, making history linear.*\
* bash
* Copy
```
git config pull.rebase true   # Set rebase as the default strategy (optional)
git pull origin master        # Rebase local changes on top of remote
```
    • Use this if: You want a cleaner history (no merge commits).
    • Warning: Rebasing rewrites history—avoid this if others work on the same branch.

***Option 3:*** *Fast-Forward Only (Error if Branches Diverge)*\
*This only allows pulls if the local branch can be fast-forwarded (no divergent commits).*
* bash
* Copy
```
git config pull.ff only       # Allow only fast-forwards (optional)
git pull origin master        # Will fail if branches diverged
```
    • Use this if: You want strict control and prefer resolving conflicts manually.


### Tags :
*anytime in project you can set a tag to project as its version. For example now we think we have a reliable and releasable condition on out project we can give attach a tag that it is new version of our product.*\ 
*Using command :*\
`git tag -a v1.0 -m ‘a message about this tag’`\
*which v1.0 is the name of tag*

*even we can attach tags to before commits . For example we can choose a commit from `git log` and copy part of its hash name from its start and (for example `f9cd629f28c1e2eb7b2666a33d3d88b0a3a0d8f0` ) and use command below :*\
`git tag -a v0.5 f9cd629f28c1e2e -m ‘some message’`\
*this will create a tag version for exactly after that commit. *

*Now we can `checkout` to each tag and see the files in that version using command below :*\
`git checkout v1.0`

*but we cant change that version*

*we can show the last commit of the tag using command below:*\
`git show v1.0`

*and we can push these tags to remote using command below :*\
`git push origin v.10`\
*or for all tags we have :*\
`git push origin --tags`

*and we can see our all tags names :*\
`git tag`

### signature ( GPG ):
*first gpg should be installed on system*\
*then create a private and public key using gpg command below :*\
`gpg -gen--key  `

*then using command below you can see your generated keys :*\
`gpg --list-keys`

*then you should edit your git config and give your key to git config as its signingKey*

*first using command below show see your private key :*\
`gpg --list-secret-keys --keyid-format LONG`
*then something like below will show you :*
```
/home/amirmohamad/.gnupg/pubring.kbx
------------------------------------
sec   rsa3072/7DC32DACB443D194 2025-04-09 [SC] [expires: 2027-04-09]
      7DD477E29AF004A3095016217DC32DACB433D194
uid                 [ultimate] amirmohamad <amir.mdtf@gmail.com>
ssb   rsa3072/A09913EE2825B0DE 2025-04-09 [E] [expires: 2027-04-09]
```
*then copy the `7DC32DACB443D194` part*

*then using command below give this key to git as its signingKey :*

`git config --global  user.signingKey 7DC32DACB443D194`

*now you can sigh your tags and commits using commands below :*

*for signing tags isntead of :*\
`git tag -a v1.0 -m ‘text’`\
*use :*\
`git tag -s v1.0 -m ‘text’`

*and to sign commits instead of :*\
`git commit -m ‘text’`\
*use :*\
`git commit -S -m ‘text’`

*when a tag is signed use `git show v1.0` command ( for `tag v1.0`) to see it is signed 
and if others need to verify it they should have the public key and use command below :*\
`git tag -v v1.0`


### git blame :
*to see who last worked on a file or exact lines of a file*\
`git blame index.php -L4,5`\
*the above command will show who worked last on lines 4 to line 5*

*help : for example for see `help` about `blame` use `git help blame`*

### git bisect :
*this will help you find when the bug created on your project*

#### commands 

*first go to top level directory of project ( mine is `/tmp/project` )*\
*then :* `git bisect start`\
*then you should use `git log` command to copy the hash name of the `commit` which in that state project didn’t have the bug*\
*then type command `git bisect bad`*\
*which means in current state we have bug*\
*then use the hash copied before and use command below:*\
`git bisect good ce703bd8c5696e1feafd31f632d65071880d65a4`\
*which means in that state we had no bug*\
*then it will go one by one to commits till now to let you test if you have bug any time you have bug use :*\
`git bisect bad`\
*and anytime there is no bug use :*\
`git bisect good`







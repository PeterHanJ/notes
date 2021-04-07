#### Moving a project from Subversion to Git

Now that you are convinced you should move your project out of Subversion and into Git there are a few steps you need to take so that you can save history.

First, you need to make sure each user in the auth.conf file in Subversion has a line in an empty text file that links to there name and email address.

Sample auth.conf file:
`JMOSES=jomama,JOMAMA`
`NDOGG=ruggster,RUGGSTER`

Matching authors.txt file:
`jomama = John Moses <moses.john.r@gmail.com>`
`JOMAMA = John Moses <moses.john.r@gmail.com>`
`ndogg = Nate Dogg <nate.dogg@arecordcompany.com>`
`NDOGG = Nate Dogg <nate.dogg@arecordcompany.com>`

Second, create a directory to store your new repo. `mkdir project && cd project`

Create a new git repo `git init`

Then checkout the project from subversion, passing –no-metadata is actually recommended against but it worked for me so I am posting it here. `git svn init http://svn.com/proj --no-metadata`

Pass your new authors file to match up users, if done properly your Github user account will now show your contributions
`git config svn.authorsfile ~/authors.txt`

And then bring it home
`git svn fetch`

I actually had to run fetch a few times as I was burned by the case sensitivity of the auth.conf/authorsfile, which is why I have multiple lines for different users.

Running `git svn fetch` actually saves your progress too, which is nice for large projects.





```git
git init  //初始创建
git add --all//添加到index
git commit -m "Initail Commit" //提交

git remote -v   //查看远程分支
git remote add origin https://bitbucket.sg.uobnet.com/scm/uob.git   //添加指定远程仓库
git remote remove origin //移除远程分支
git push -u origin master  //将本地master分支push到远程仓库
```


# svn2git
backup working svn repository to remote git server, keeping commit message


## Init

We use svn project name "mip" as sample
First get all author mapping list
```bash
cd svn/project/trunk/mip
svn log --xml | grep author | sort -u
```
edit users.txt

then clone from svn
```bash
git svn clone http://your_svn_server/your_svnroot/project/trunk/mip --authors-file=users.txt --no-metadata
```

let say you want to keep backup at https://YourName@bitbucket.org/YourName/mip.git

```bash
git remote add origin  https://YourName@bitbucket.org/YourName/mip.git
git push -u origin --all
git push -u origin --tags
```

## Update

You have commit something into svn, now you want to backup to git server. 
First you run ```bash svn up``` and ```bash svn diff``` to make sure local workspace is up to date

get last revision of svn, sxxxxxx
```bash
svn up | tail -1 | cut -d ' ' -f3 | tr -d '.'
```

get last revision git have, gxxxxxx
```bash
tail -1 .git/logs/refs/remotes/git-svn | cut -f2 | tr -d 'r'
```

now reverse workspace to git point:
```bash
svn merge -r sxxxxx:gxxxxx .
```

You are ready to sycn local git master
```bash
git svn fetch
git rebase remotes/git-svn master
```

finally push to remote
```bash
git push -u origin
```

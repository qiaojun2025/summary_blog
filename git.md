## 命令
```
git config
git init
git clone
git add
git reset
git commit
git status
git push 
git pull 
git branch
git merge
git log
git stash
git revert
git rm
```

## 示例

### local file add

```
git init demo1
git add test1.text
git status
git commit -m "add test file"


```

### remote file add

```
git clone https://
git pull origin master
git push
```



## 本地git流程

编写代码 -- 保存变更 add -- 提交变更 commit -- 推送变更 push  

## Branch

```
git branch b1
git branch --list
git checkout b1
    some action on b1
git checkout master
    some action on master
git merge b1
git branch -d b1
```

## 合并与冲突

### 不同分支的冲突

```
git branch b2
git checkout b2 
    edit one file, then add and commit the file
git checkout master
    edit one file, then add and commit the file
git diff master b2

git merge b2
    manu edit the conflict file, then add and commit
```

### 同一个分支的冲突

```
git pull --rebase origin master
git mergetool
git rebase --continue
git push origin master
```

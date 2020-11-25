# :zap: 1. 关于仓库
初始化仓库
```
git init
````
忽略权限
```
git config core.fileMode false
```
remote
```
git remote rm origin xxx
git remote add origin2 xxx
git remote -v
git remote set-url origin git://new.url
```

# :zap: 2. 关于分支
查看所有分支
```
git branch -a
git branch -av
```

删除分支
```
删除本地的本地分支：
git branch -d xxx
删除本地的远程分支：
git branch -r -d origin/xxx
推送本地的远程分支：
git push origin :xxx
```

从远程更新分支列表
```
git remote update origin --prune
git branch -d xxx
没有merge的分支需要用 -D
```

重命名分支
```
删除远程分支：
git push -d origin xxx
重命名本地分支：
git branch -m xxx yyy
推送本地分支：
git push origin yyy
```

推送/拉取所有分支
```
git push origin2 -all
git pull -all
```

# :zap: 3. 关于提交
合并其他分支某次提交
```
git cherry-pick commit_id_b commit_id_a （根据cherry-pick的顺序来排序）
```

去掉中间某次提交（去掉后从其他分支仍然能merge过来）
```
git rebase -i commit_id （commit_id为想要删除的commit之前的commit_id）
pick xxx 修改为 drop xxx，然后保存
git push -f
```

修改代码但不产生新的commit记录
```
场景一：还没有push
git add .
git commit -m "xxx"
(Code)
git add .
git commit --amend，然后修改保存
git push
```
```
场景二：已经push
git add .
git commit -m "xxx"
git push
(Code)
git add .
git commit --amend，然后修改保存
git push -f
```

合并历史的commit为一个
```
git rebase -i commit_id（commit_id为想要删除的commit之前的commit_id）
不连续的移动到一起，并把需要合并的除了第一个以外其他的 pick xxx 改成 s xxx，然后保存
git push -f
```

本地和线上回到以前某次提交
```
git reset --hard xxx
git push -f
```

查看某个文件修改历史
```
git log -p filename 查看某个文件修改详情
git log filename 查看某个文件修改提交历史
git show commit_id filename 查看某次提交内的某个文件的修改详情
```

# :zap: 4. .gitignore文件
排除已忽略部分的部分文件/部分文件夹
```
/public/*
!/public/assets/
/public/assets/*
!/public/assets/administration
/public/assets/administration/*
!/public/assets/administration/v1
/public/assets/administration/v1/*
!/public/assets/administration/v1/shared
/public/assets/administration/v1/shared/*
!/public/assets/administration/v1/shared/kindeditor
```

# :zap: 5. 其他操作

```
git rebase branch_name 把branch_name分支代码挪到当前分支作为基准（当前分支代码在最后）
git pull 相当于git fetch + merge，git pull --rebase 相当于 git fetch + rebase
git push 不加参数时，根据当前HEAD来定位。
git push origin node:branch_name
git fetch origin branch_name / git fetch origin node:branch_name
git checkout branch_name^^^ = git checkout branch_name~3，移动HEAD节点，git checkout HEAD^2，表示移动到合并时父分支的第2个节点。
git branch -f branch_name (commit_id / 引用) 移动分支到某节点
git reset本地，git revert 添加一个提交用以撤回
git tag version commit_id 打标签
git describe (branch_name / ref) 输出结果： tag_距离_gref的commit_id
git reset --soft commit_id 本地代码需要add。git reset --hard commit_id 本地代码完全不要了。
```
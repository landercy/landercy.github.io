# 命令

## 全局配置

```bash
# list or edit global config
git config --global -l
git config --global -e

# set up name and email
git config --global user.name "NAME"
git config --global user.email "EMAIL"

# set up credential helper
git config --global credential.helper store

# git log highlight
git config --global color.ui true

# alias
git config --global alias.s status

# faster cd when using oh_my_zsh
git config --add oh-my-zsh.hide-dirty 1

# line ending
## for Mac/Unix/Linux
git config --global core.autocrlf input
git config --global core.safecrlf true
## for Windows
git config --global core.autocrlf true
git config --global core.safecrlf true

# Chinese Character Support
git config --global core.quotepath false

# filemod
git config --global core.filemode false
```

## 基本操作
```bash
# basic operation
git clone <repo_url>
git fetch --all
git pull --all

git init
git status
git add file.txt
git add.
git commit -m "COMMENT"
git push origin master

git tag
git tag TAG
git checkout TAG^
git checkout TAG~1
git tag -d TAG

git log
git checkout HASH
git checkout FILENAME
git reset HEAD FILENAME
git reset --hard TAG
git revert HEAD

# log prettify
git log --pretty=oneline
git log --pretty=oneline --max-count=2
git log --pretty=oneline --since='1 hour ago'
git log --pretty=oneline --until='5 minutes ago'
git log --pretty=oneline --author=NAME

# change the previous commit
git commit --amend -m "COMMENT"

# moving files
git mv FILE DIR

git cat-file -p HASH
git cat-file -t HASH

# braches
git checkout -b <local branch> origin/<remote branch>
git checkout BRANCH
git merge BRANCH
git rebase BRANCH 

git branch -a
git branch -av
git branch -vv
git branch -r
git branch -d <branch name>

# search for all conflicting files.
grep -lr '<<<' .

git checkout --[ours|theirs] PATH/FILE
grep -lr '<<<' . | xargs git checkout --[ours|theirs]
```


# 场景

## 切换仓库地址

```bash
# 查看远程仓库地址 
git remote -v

# 设置分支track远程分支 
git checkout -b _0000 # 需要先切到一个远程没有的分支，不然会报错：fatal: Cannot force update the current branch.
git branch -r | grep -v "\->" | while read remote; do git branch -f -t "${remote#origin/}" "$remote"; done
git checkout master && git branch -D _0000
git fetch --all && git pull --all
git branch -l

# 变更远程仓库地址
git remote set-url origin <url>
git remote -v 

# 推送所有分支到新仓库
git push --all
```

```py
#!/usr/bin/python3
import subprocess

cmd = 'find ./ -type d -name ".git" -exec dirname {} \;'
dir_list = subprocess.check_output(cmd, shell=True).decode().splitlines()

cmd = "git remote -v && git fetch --all && git pull --all"
for dir in dir_list:
    subprocess.run(cmd, shell=True, cwd=dir)
```

```py
#!/usr/bin/python3

import subprocess  
import shlex  

base_path = '/data/code'

subprocess.run(['echo', '---------------------'])
subprocess.run(['date', '+%Y-%m-%d %H:%m:%S'])
subprocess.run(['echo', '---------------------'])

cmd = 'find {} -type d -name ".git" -exec dirname {} \;'.format(base_path, '{}') 
output = subprocess.check_output(shlex.split(cmd))  
dir_list = output.decode().splitlines()

cmd = """
    git remote -v && \
    git checkout -b _0000 && \
    (git branch -r | grep -v "\->" | while read remote; do git branch -f -t "${remote#origin/}" "$remote"; done) && \
    git checkout master && \
    git branch -D _0000 && \
    git fetch --all && \
    git pull --all
"""
for dir in dir_list:
    subprocess.run(cmd, shell=True, cwd=dir)
```

## 强制更新，覆盖本地
```bash
# 1 
git reset --hard

# 2
git fetch --all && git reset --hard origin/master && git pull
```

## 代码回滚，强制覆盖远程

```bash
git reset --hard <HASH>
git push origin HEAD:master --force
```

## 移除untracked files
```bash
git clean -df
```

## 命令行中使用token
```bash
git clone https://yangyang:eXufJPUTyoXMYN4SJgBA@gitlab.linde/linde/ec-shop-php.git
```

## 优化仓库大小
```bash
git gc --prune=now  git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

## 使用私钥

```bash
# ~/.ssh/config
Host HOST_IP
Port PORT
User USERNAME
IdentityFile ~/.ssh/id_rsa
```

### 报错

#### permission denied (public key)

密钥正确的情况下，在clone gerrit项目时提示没有权限，

原因是OpenSSH没有启用ssh-rsa

把rsa算法改成ed25519可以解析问题

```bash
ssh-keygen -t ed25519 -C <email>
```

Config 文件更换密钥
```ini
IdentityFile ~/.ssh/id_ed25519
```

# 报错

## SSL certificate problem: self signed certificate

> fatal: unable to access 'https://xxx.git': SSL certificate problem: self signed certificate

```bash
git config --global http.sslVerify false
```

## Exiting because of unfinished merge

>hint: Please, commit your changes before merging.
>
>fatal: Exiting because of unfinished merge.
```bash
git reset --hard
```

# 其他

## 各种配置

```bash
# 只要有一个命令被模糊匹配到了，Git 会自动运行该命令。
# 5s的反悔时间。
help.autocorrect 50


# 外部比较和合并工具
diff.external
merge.tool

# white space
core.whitespace blank-at-eol,blank-at-eof,space-before-tab,-indent-with-non-tab,-tab-in-index,-cr-at-eol
```


# Alias

```bash
alias gco='git checkout'
alias gcd='git checkout $(git_develop_branch)'
alias gf='git fetch'
alias gfa='git fetch --all --prune --jobs=10'
alias gl='git pull'
alias gb='git branch'
alias ggsup='git branch --set-upstream-to=origin/$(git_current_branch)'

alias gsb='git status --short --branch'
alias gaa='git add --all'
alias gcam='git commit --all --message'
alias gp='git push'
alias gpsup='git push --set-upstream origin $(git_current_branch)'

alias gd='git diff'
alias gds='git diff --staged'
alias gdup='git diff @{upstream}'

alias grset='git remote set-url'
alias grv='git remote --verbose'

alias grhh='git reset --hard'
```


grv
grset https://repo.lindemh-cn.com/gitlab/customerapp/customerapp/$(basename $(pwd)).git
grv
gp --all
cd ..

git pull -u origin master

# fatal: refusing to merge unrelated histories

--allow-unrelated-hhistories 不管用

# fatal: CRLF would be replaced by LF

修改文件换行符为 LF


# 查看指定文件提交的修改内容
```bash
# N: 数字
git log -p -<N> <FILE_NAME>
```

# 未验证

在Git中，你可以使用git hooks来禁止指定分支合并到master分支。你可以在.git/hooks/pre-receive文件中添加以下代码：

#!/bin/bash  
  
while read oldrev newrev refname; do  
    if [[ "$refname" == "refs/heads/master" ]]; then  
        protected_branch='test'  
        if git branch --contains $newrev | grep -qE "^(\* )?$protected_branch$"; then  
            echo "You cannot merge the '$protected_branch' branch into the 'master' branch."  
            exit 1  
        fi  
    fi  
done  
 
这段代码会检查提交到master分支的变更，如果这些变更包含了被保护的分支（例如'test'分支），则会阻止这些变更被合并到master分支。

请注意，这段代码必须被添加到每个开发者的本地仓库中。你可以将这段代码复制到一个文件中，然后将该文件保存为.git/hooks/pre-receive。然后，使用chmod命令将该文件设置为可执行文件：

chmod +x .git/hooks/pre-receive  
 
这样，每当有人尝试将被保护的分支合并到master分支时，就会收到一条错误消息，并且变更不会被合并到master分支中。

你可以将上述代码添加到你的GitLab项目的“Custom Git Hooks”中，以便在GitLab服务器上执行该代码，而不需要在每个开发者的本地仓库中添加代码。以下是如何将该代码添加到GitLab的步骤：
打开你的GitLab项目并导航到“Settings”页面。
选择“Repository”选项卡，然后向下滚动到“Custom Git Hooks”部分。
在“Custom Git Hooks”部分，选择“Pre-receive hooks”。
在“Pre-receive hooks”中，将上述代码复制并粘贴到文本框中。
点击“Add pre-receive hook”按钮以保存代码。

这样，当有人尝试将被保护的分支合并到master分支时，该代码将在GitLab服务器上执行，并阻止变更被合并到master分支中。请注意，你必须是项目的所有者或管理员才能添加自定义Git钩子。
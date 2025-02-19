# 远程仓库

Git 作为分布式的版本管理系统，每个终端都有自己的 Git 仓库。但团队协作还需要一个中间仓库，作为中心进程同步各个仓库。于是服务器（远程）仓库就来承担这个职责，服务端不仅有仓库，还有配套的相关管理流程。我们可以使用公共的 Git 服务器，也可以自己搭建一套 Git 服务器。

![](../assets/git/remote_repository_1.png)

- 公共的 Git 服务器有：[Github](https://github.com/)、[Gitlab](https://about.gitlab.com/)、[码云 Gitte](https://gitee.com/) 等
- 搭建的私有 Git 服务器，如开源的 Gitlab、[Gitea](https://github.com/go-gitea/gitea)等

## 远程用户登录

Git 服务器一般提供两种登录验证方式：

- HTTS：基于 HTTPS 连接，使用用户名、密码身份验证
    - 每次都要输入用户名、密码，当然可以记住
    - 地址形式：`https://github.com/xjjeffery/learn_git.git`
- SSL：采用 SSL 通信协议，基于公私钥进行身份验证，所以需要额外配置公私秘钥
    - 不用每次输入用户名、密码，比较推荐的方法
    - 地址形式：`git@github.com:xjjeffery/learn_git.git`

### 远程用户登录：HTTS

基于 HTTPS 的地址连接远程仓库，Github 的仓库克隆、拉取是不需要验证的。

```bash
$ git clone 'https://github.com/github/gitignore.git'
Cloning into 'gitignore'...

# 仓库配置文件“.git/config”
[remote "origin"]
  url = https://github.com/github/gitignore.git
  fetch = +refs/heads/*:refs/remotes/origin/*
  pushurl = https://github.com/github/gitignore.git
```

推送代码的时候就会提示输入用户名、密码了，否则无法提交。记住用户密码的方式有两种：

- URL 地址配置：在原本 URL 地址上加上用户名、密码，`https://后加用户名:密码@`

```bash
# 直接修改仓库的配置文件“.git/config”
[remote "origin"]
  url = https://用户名:密码@github.com/xjjeffery/learn_git.git
  fetch = +refs/heads/*:refs/remotes/origin/*
  pushurl = https://github.com/xjjeffery/learn_git.git
```

- 本地缓存：创建一个缓存文件 `.git-credentials`，存储输入的用户名、密码。

```bash
# 参数 “--global” 全局有效，也可以针对仓库设置 “--local”
# store 表示永久存储，也可以设置临时存储
git config --global credential.helper store

# 存储内容如下，打开文件“仓库 \.git\.git-credentials”
https://xjjeffery:xxxx@github.com
```

### 远程用户登录：SSH

SSH（Secure Shell，安全外壳）是一种网络安全协议，通过加密和认证机制实现安全的访问和文件传输等业务，多用来进行远程登录、数据传输。SSH 通过公钥、私钥非对称加密数据，所以 SSH 需要生成一个公私钥对，公钥放服务器上，私钥自己留着进行认证。

生成公私钥：通过命令生成公私钥，在此过程中一直回车即可完成。

```bash
ssh-keygen -t rsa -C
```

Linux 是在 `~/.ssh` 目录下，Windows 在 `C:\Users\<user name>\.ssh` 目录下，文件 `id_rsa.pub` 的内容就是公钥。

配置公钥：打开 `id_rsa.pub` 文件，复制内容。Github上，打开 Setting -> SSH and GPG keys -> SSH keys -> 按钮 New SSH key，标题(Title)随意，秘钥内容粘贴进去即可。

![](../assets/git/remote_repository_2.png)

SSH 配置完后，可用 `ssh -T git@github.com` 来检测是否连接成功。

## 操作指令

| **指令** | **描述** |
| --- | --- |
| `git clone <remote_repo>` | 从远程仓库克隆到本地(当前目录) |
| `git clone -b <branch> <remote_repo>` | 从远程仓库克隆指定的分支到本地 |
| `git remote -v` | 查看所有远程仓库，不带参数 -v 只显示名称 |
| `git remote show <remote>` | 显示某个远程仓库的信息 |
| `git remote add <name> <url>` | 增加一个新的远程仓库，并命名 |
| `git remote rename <old> <new>` | 修改远程仓库名称 |
| `git remote rm <remote-name>` | 删除远程仓库 |
| `git pull <remote> <branch>` | 拉取远程仓库的变化，并与本地版本合并 |
| `git pull` | 同上，针对当前分支 |
| `git pull --rebase` | 使用 `rebase` 的模式进行合并 |
| `git fetch <remote>` | 获取远程仓库的所有变动到本地仓库，不会自动合并！需要手动合并 |
| `git push` | 推送当前分支到远程仓库 |
| `git push <remote> <branch>` | 推送本地指定分支到远程仓库的指定分支 |
| `git push <remote> --force/-f` | 强行推送当前分支到远程仓库，即使有冲突，很危险 |
| `git push <remote> --all` | 推送所有分支到远程仓库 |
| `git push –u` | 参数 `–u` 表示与远程分支建立关联，第一次执行的时候用，后面就不需要了 |
| `git push <remote> -d <branch>` | 删除远程仓库上的某个分支 |

!!! warning "注意"

    操作远程仓库时，以下要点需严格遵守：

    - 每次推送前，先将对应的远程分支 `pull` 到本地分支，即将其他开发者推送的更新合并到本地分支中，确保本地修改与远程分支中的更新没有冲突
    - 禁止对公共分支（例如 `dev`）进行 `rebase` 操作。可以对本地分支进行 `rebase` 操作用于简化提交记录
    - 禁止对公共分支进行 `commit --amend` 操作

    一个重要原则就是严禁任何修改公共分支提交历史的操作！`rebase` 操作只在非公共分支上进行，主要用来清理提交记录（将多个零散对提交记录合并为一个提交）。

## `fetch` 与 `pull` 的不同

两者都是从服务端获取更新，主要区别是 `fetch` 不会自动合并，不会影响当前工作区内容。

> `git pull` = `git fetch` + `git merge`

- 如下面图中，`git fetch` 只获取了更新，并未影响 `master` 、`HEAD` 的位置。
- 要更新 `master`、HEAD 的位置需要手动执行 `git merge`合并。

![](../assets/git/remote_repository_3.png)

```bash
# fetch 只更新版本库
$ git fetch
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), 663 bytes | 44.00 KiB/s, done.
From github.com:xjjeffery/learn_git
   2ba12ca..c64f5b5  main       -> origin/main

# 执行合并，合并自己
$ git merge
Updating 2ba12ca..c64f5b5
Fast-forward
 README.md | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```
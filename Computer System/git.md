# Git

- [Git](#git)
  - [基础](#基础)
    - [各种配置](#各种配置)
  - [操作](#操作)
  - [Git reset](#git-reset)
    - [Git reset 不指定文件路径](#git-reset-不指定文件路径)
    - [Git reset 通过路径来重置](#git-reset-通过路径来重置)
    - [压缩历史记录](#压缩历史记录)
  - [搜索项目等等操作](#搜索项目等等操作)

## 基础

### 各种配置

去这个网站`https://www.ipaddress.com/`查询

- github.com
- github.global.ssl.fastly.net

后面的两个网站  将ip如这种格式 加入hosts里面, ``C:\Windows\System32\drivers\etc``;因为Hosts无法保存拖到桌面用txt打开,linux同样如此.

- git 登陆用token idea    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
- gitee token   730ceedc144839080fae255ae9313050
- Yang git ssh   git@github.com:yangduojin/book.git
- Gitee ssh CxEWnzHF8dJl3PnmYgFn08PHMRqrRXFj9mt2vwGrySU

git配置代理v2rayN

```conf
git config --global https.proxy http://127.0.0.1:10809  
git config --global https.proxy https://127.0.0.1:10809
git config --global http.proxy 'socks5://127.0.0.1:10808'
git config --global https.proxy 'socks5://127.0.0.1:10808'
git config --global --unset http.proxy
git config --global --unset https.proxy
# ssl出问题的时候使用这个
git config --global http.sslCAInfo "F:\Program Files\Git\mingw64\ssl\cert.pem"
```

在C:\Users\90362 目录下用gitbush执行这个命令 ，ssh-keygen -t rsa -C yangxsilent@gmail.com 三次回车，生成公钥私钥,公钥内容放入git的SSH and GPG keys里面，git pull的时候在面后 加上 ssh的链接

## 操作

1. 本地仓库操作
   - git config --global user.name "maweiqi"
   - git config --global user.email "hello@maweiqi.cn"
   - git config --list
   - git config user.name
   - git init
   - git status
   - git status –s     ,(short)
   - git add .
   - git add login.java
   - git reset 暂存区文件取消暂存

   // 将暂存区的文件修改提交到本地仓库

   - git commit -m '版本描述'
   - git commit -am "版本描述"     ,(add + message)
   - git log / reflog
   - git reset --hard 版本号 (log/reflog的版本号)
   - .gitignore
     - *.a (被忽略)
     - !lib.a  (取反,会被管理)
     - /TODO  (TODO文件会被忽略)
     - build/  (build/ 目录下面的所有文件会被忽略)
     - doc/*.txt
     - doc/**/*.pdf
   - git rm 文件名
   - git commit -m '删除描述'
   - 没有删除权限
     - git add .
     - git commit -m 'xxxx'
     - git rm -f 要删除文件的名字
2. 克隆远程仓库到本地( 不需要.git )
   - git clone `https://gitee.com/ithe/myRepo3.git`
   - git remote     (查看当前目录跟远程仓库有没有建立连接)
   - git remote -v
3. 本地仓库跟远程建立连接
   - git remote add origin 远程仓库地址 (本地仓库跟远程仓库建立联系)如果当前仓库跟别的仓库有连接，可以去config里面改
   - git remote rm origin   origin 建立连接后默认仓库名
   - git push [origin master]
   - Git fetch origin master 拉去最新不合并
   - git pull [origin master] 拉取的时候并不会覆盖自己的新代码，而是 origin maste 这个分支，并不是所有的分支
   - 代码合并，本地建立一个分支 专门用来pull远程的最新版，将自己本地分支设置为dev，本地合并代码，再上传到github

4. Git 分支
   - git branch
   - git branch -r
   - git branch -a
   - git branch 分支的名字
   - git checkout 分支的名字
   - git checkout -b 本地分支名 origin/远程分支名
   - git push origin 分支的名字
   - git merge 分支的名字
   - git branch -d 要删除的分支名字
   - git branch -D 要强行删除的分支名字
   - git push origin –d 要删除的远程分支的名字

5. Git idea 使用
   - File --> Settings --> Version Control下的git选项：
   - vcs create git repository
   - idea add commit push
   - 出现问题去config配置文件改链接
   - 如果推送的时候出现权限拒绝
   - 在要push的项目 打开git黑窗口输入
   - git pull origin master --allow-unrelated-histories
   - git push -u origin master -f
   - 再对项目右键 git repository remotes添加远程仓库
   - 可以pull远程 同步开发
   - 可以合并分支 在主分支 点击分支 merge into currents
   - 左下角 version controller log可以切换版本checkout version
   - 使用总结
     1. 拉取下来的代码新建一个分支dev专门在dev上面开发
     2. dev开发完提交之后去test分支,先拉取最新的代码,然后一定要在test分支合并本地的dev->merge into current
     3. 然后修改冲突提交到远程(如果冲突了也是修改dev,test这个分支不要修改),提交远程是提交test
     4. 再去dev分支 点击test->rebase current onto selected

## Git reset

### Git reset 不指定文件路径

- Git reset --soft HEAD~ 将repo的回退一个版本
- Git reset [--mixed] HEAD~ 将repo回退一个版本，并将该版本的文档推到缓存区。工作区保持原样(修改或未修改)
- Git reset --hard HEAD~ 将repo回退一个版本，并将该版本推到另外两个区域，三个区域版本一致
- reset 命令会以特定的顺序重写这三棵树，在你指定以下选项时停止：
  1. 移动 HEAD 分支的指向 （若指定了 --soft，则到此停止）
  2. 使索引看起来像 HEAD （若未指定 --hard，则到此停止）
  3. 使工作目录看起来像索引  --hard

### Git reset 通过路径来重置

指定了一个路径，reset 将会跳过第 1 步，并且将它的作用范围限定为指定的文件或文件集合。不过索引和工作目录 可以部分更新，所以重置会继续进行第 2、3 步。

- git reset [--mixed HEAD] file.txt(还具有取消暂存文件)
  - 移动 HEAD 分支的指向 （已跳过）
  - 让索引看起来像 HEAD （到此处停止）
- git reset eb43bf file.txt: 将对应版本的文件放到缓存区，替换之前缓存区的版本，工作目录和repo的版本还是最新版

### 压缩历史记录

a - b - c - d 四个版本bc有问题,d没问题;head是d，index d,working directory d

- git reset --soft HEAD~3
  - head是a, index d, working directory d
- git commit
  - a - d (中间的历史记录被压缩了)
  - head d, index d, working directory d

|                            | HEAD | Index | Workdir | WD Safe? |
| -------------------------- | ---- | ----- | ------- | -------- |
| Commit Level               |
| reset --soft [commit]      | REF  | NO    | NO      | YES      |
| reset [commit]             | REF  | YES   | NO      | YES      |
| reset --hard [commit]      | REF  | YES   | YES     | NO       |
| checkout \<commit>         | HEAD | YES   | YES     | YES      |
| File Level                 |
| reset [commit] \<paths>    | NO   | YES   | NO      | YES      |
| checkout [commit] \<paths> | NO   | YES   | YES     | NO       |

- "REF" 表示该命令移动了 HEAD 指向的分支引用
- "HEAD" 则表示只移动了 HEAD 自身
- WD Safe? 如为 NO，那么运行该命令之前请考虑一下。

checkout 相当于操作三棵树

1. 不带路径
   - git checkout [branch] 与 git reset --hard [branch] 类似，会更新所有三棵树使其看起来像 [branch]，不过有两点重要的区别。
   - checkout 对工作目录是安全的，通过检查来确保不会将已更改的文件弄丢。它会在工作目录中先试着简单合并一下，这样所有还未修改过的文件都会被更新。 而 reset --hard 则会不做检查就全面地替换所有东西。
   - checkout 如何更新 HEAD。 reset 会移动 HEAD 分支的指向，而 checkout 只会移动 HEAD 自身来指向另一个分支。两个分支master 和 develop，我们在develop 上，则head指向它，
     - 运行git reset master， develop则改变成和master指向同一个提交引用，
     - 运行git checkout master ，develop 不会移动，HEAD 自身会移动。 现在 HEAD 将会指向 master。
2. 带路径
   - 运行 checkout 的另一种方式就是指定一个文件路径，这会像 reset 一样不会移动 HEAD。 它就像 git reset [branch] file 那样用该次提交中的那个文件来更新索引，但是它也会覆盖工作目录中对应的文件。 它就像是 git reset --hard [branch] file（如果 reset 允许你这样运行的话）， 这样对工作目录并不安全，它也不会移动 HEAD。
   - 此外，同 git reset 和 git add 一样，checkout 也接受一个 --patch 选项，允许你根据选择一块一块地恢复文件内容。

## 搜索项目等等操作

1. 常用词
     - watch：会持续收到该项目的动态
     - fork：复制其个项目到自己的Github仓库中
     - star，可以理解为点赞
     - clone，将项目下载至本地
     - follow，关注你感兴趣的作者，会收到他们的动态

2. in限制搜索 in关键词限制搜索范围
     - 公式 ：xxx(关键词) in:name或description或readme
       - xxx in:name 项目名包含xxx的
       - xxx in:description 项目描述包含xxx的
       - xxx in:readme 项目的readme文件中包含xxx的组合使用
     - 组合使用
       - 搜索项目名或者readme中包含秒杀的项目
       - xxx in:name,readme

3. star和fork范围搜索
     - 公式：
       - xxx关键字 stars 通配符 :> 或者 :>=
       - 区间范围数字： stars:数字1…数字2
     - 案例
       - 查找stars数大于等于5000的springboot项目：springboot stars:>=5000
       - 查找forks数在1000~2000之间的springboot项目：springboot forks:1000…5000
     - 组合使用
       - 查找star大于1000，fork数在500到1000的springboot项目：springboot stars:>1000 forks:500…1000

4. awesome搜索
     - 公式：awesome 关键字：awesome系列，一般用来收集学习、工具、书籍类相关的项目
     - 搜索优秀的redis相关的项目，包括框架，教程等 awesome redis

5. #L数字 (highlight)
     - 一行：地址后面紧跟 #L10
       - `https://github.com/abc/abc/pom.xml#L13`
     - 多行：地址后面紧跟 #Lx - #Ln
       - `https://github.com/moxi624/abc/abc/pom.xml#L13-L30`

6. T搜索
    - 在项目仓库下按键盘T，进行项目内搜索

7. 搜索区域活跃用户
      - location：地区
      - language：语言
      - 例如：location:beijing language:java

[更多github快捷键 原文链接](https://docs.github.com/en/get-started/using-github/keyboard-shortcuts)

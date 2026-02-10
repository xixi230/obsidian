回顾一下，前面我们已经了解了本地Git仓库的原理

git作为分布式管理系统，我们自然需要在不同的设备进行开发，这往往需要部署一个远程仓库例如GitHub？

NO，也许你甚至不需要一个github，你只需要我们刚刚讲解的ssh服务器。

如果你是一个忠实的`代码掌控爱好者`你希望所有的代码和服务都是自己可控的，也许你希望自己git远程仓库也是自己可以掌控的

https://git-scm.com/book/en/v2/Git-on-the-Server-The-Protocols

在git的官方在线书籍中，我们可以探究github的工作原理到底是什么

![[Pasted image 20260208120336.png]]

引言揭示了git服务器，远程仓库往往是一个`裸仓库`
`(bare  repo)`

它是一个没有工作区的git仓库
bare 仓库 = 只有 `.git` 目录里的内容，没有工作区
同时也没有暂存区(bare 仓库不做 commit 操作，只接收 push 和响应 fetch/clone)
![[Pasted image 20260208122812.png]]

Git 可以使用四种不同的协议来传输数据：本地协议、HTTP 协议、安全外壳 (SSH) 协议和 Git 协议

这里我们推荐使用SSH协议进行传输
![[Pasted image 20260208122953.png]]

SSH的优点在于他是安全的，所有的信息都是加密的，同时和HTTPS一样高效

当然，由我们前面学习到的，ssh作为协议及其适合个人开发和小团队开发，但是不适合开源



登陆到我们的ssh服务器
```bash
ssh user@your-server.com
```

进入后创建一个bare仓库：
```bash
mkdir -p ~/repos/my-project.git
cd ~/repos/my-project.git
```

和我们上节课讲的一样，我们需要初始化git仓库
```bash
git init --bare hello
```

![[Pasted image 20260208134531.png]]

实际上这就是一个普通的.git仓库


假设我们已经处理好了远程仓库了，我们现在需要将本地仓库和远程仓库进行连接(本地仓库记得也要初始化)：

```bash
git remote add origin user@your-server.com:repos/my-project.git
```

#### SSH URL 格式
```bash
user@host:path
```

```bash
git remote add origin user@your-server.com:repos/my-project.git
│    │      │   │      │    │               │
│    │      │   │      │    │               └── 服务器上的路径
│    │      │   │      │    └── 服务器地址
│    │      │   │      └── SSH 用户名
│    │      │   └── 远程仓库的别名（惯例叫 origin）
│    │      └── 添加一个远程
│    └── 管理远程仓库的子命令
└── git 命令
```

- `user@` — SSH 登录用户
- `your-server.com` — 服务器地址（域名或 IP）
- `:` — 分隔符
- `repos/my-project.git` — 仓库路径（相对于用户 home 目录)


现在我们已经建立了远程仓库和本地的连接，我们可以尝试使用ssh语法从这个远程仓库复制一个文件出来
``` 
scp user@your-server.com:repos/my-project.git/HEAD .
```
![[Pasted image 20260208152349.png]]

查看本地仓库：
```bash
ls
```
![[Pasted image 20260208152434.png]]
确实直接复制过来了


有趣的是Git原生支持SSH URL，所以我们不妨可以以后直接使用git命令进行本地和远程之间的传输
```bash
git clone user@your-server.com:repos/my-project.git
```
![[Pasted image 20260208152849.png]]
我们可以进入仓库
![[Pasted image 20260208152944.png]]
实际上我们可以看到远程仓库`my-project`已经被我们拉取到本地了

我们可以在本地开发修改好文件
![[Pasted image 20260208154426.png]]

接下来我们就回到了昨天的正常操作了
![[Pasted image 20260208154603.png]]

接下来直接推送
![[Pasted image 20260208154746.png]]
```bash
git push
```

有的同学可能见过`git push origin main`

实际上`origin`是你默认的远程仓库的别名，你也可以同时添加多个远程仓库，例如
```bash
git remote add origin git@github.com:你/项目.git
git remote add aliyun root@阿里云:/root/repos/my-project.git
git remote add backup git@gitee.com:你/项目.git
```

这样子你就有三个了，你想推送的时候则需要输入：
```bash
git push origin main
git push aliyun main
git push backup main
```

main代表推送当前到远程仓库的main分支
如果想指定本地仓库的master分支到远程仓库的main分支，可以写成：
```bash
git push origin master:main
```

在本地输入
```bash
git remote -v
```
可以查看本地仓库关联的远程仓库
![[Pasted image 20260208163015.png]]


回到远程仓库我们可以看看如何了:

![[Pasted image 20260208165104.png]]

非常好的推送过来了！

由于是裸仓库，没有工作目录，我们是看不到文件的，文件的源代码被压缩后放在了版本库里。

如果你想确认，可以输入
```bash
git show master:main.c
```

![[Pasted image 20260208165702.png]]

以上就是远程仓库的原理



### Github
正如上面所说，github只是微软的提供的平台，但是在我们平时使用鼠标在可视化界面的背后的原理一模一样，他只是给你套了一层web界面而已


在github创建一个新的仓库
![[Pasted image 20260208170009.png]]

你可以选择将仓库设为公开或者私密

![[Pasted image 20260208170110.png]]

完成后你将进入这样一个界面，他会给你一个连接，在自己的电脑上你可以选择SSH
![[Pasted image 20260208170828.png]]
其中SSH的这个URL格式我们已经熟悉的不能再熟悉了

==**这是一个名为git的用户，在github.com的ssh服务器上去访问shiro123444/test.git的裸仓库**==

甚至我们可以去尝试连接一下这个ssh服务器（坏
```bash
ssh git@github.com
```
![[Pasted image 20260208171517.png]]

成功通过了认证，但是github不提供shell
这就是微软不想让你知道的事情（）

我们来将我们的本地仓库和github仓库建立远程连接
```bash
git remote add github-repo-name git@github.com:/your-account/repo-name.git
```
![[Pasted image 20260208171920.png]]
注意，之前我们已经和自己的SSH服务器建立了连接，并且赋予了默认的`origin`名称，我们如果此时还是使用`origin`就会报错

跟上潮流我们可以重命名master为main分支在本地：
```bash
git branch -M main
```

![[Pasted image 20260208172612.png]]

```bash
git push your-repo-name main
```
直接推送
![[Pasted image 20260208184611.png]]
由于我们本地还没有储存github的ssh认证信息或者账号和仓库不一致就会报错

这时候我们需要生成ssh的key来绑定认证
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_shiro -C "shiro123444"
           ↓          ↓                  ↓
        算法类型    密钥保存路径       注释(随便写)
```
![[Pasted image 20260208191522.png]]

输出会有一个fingerprint，复制那一串，粘贴到github的sshkey绑定里
https://github.com/settings/keys
![[Pasted image 20260208191640.png]]


如果你有多个github账号，你可以在ssh配置里面写上别称，这样更好管理
![[Pasted image 20260208191738.png]]

>注意，如果你生成的密钥是自定义名称的，请也在.ssh的.config里配置好，因为ssh默认会在.ssh寻找`~/.ssh/id_rsa`、`~/.ssh/id_ed25519`，如果寻找失败就会报错


### 问题排查
使用`ssh -T git@github.com`

- 如果返回 `Hi username! You've successfully authenticated...` → SSH 没问题，检查远程 URL
- 如果返回 `Permission denied (publickey)` → 密钥没对上，大概率就是上面说的第一个原因


可以再一步详细看看输出
```bash
ssh -vT git@github.com
```

也可以进一步输出到文件
```bash
ssh -vT git@github.com 2> output.log
```

![[Pasted image 20260209105206.png]]
以上输出则表示SSH搜索了整个.ssh文件夹的默认密钥，但是他不会主动尝试你自己的自定义密钥，需要你在.config手动配置确保安全


将本地仓库ssh的url更新
```bash
git remote set-url shiro github-shiro:shiro123444/test.git
```
这里由于设置了别名，github-shiro相当于`git@github.com`,同时附带上了密钥的位置，告诉ssh这里需要用这个密钥而不是其他密钥进行传输认证


再次push即可
![[Pasted image 20260208192102.png]]


刷新可以看到github上已经更新了：

![[Pasted image 20260208192156.png]]



### Github codespace

github提供了codespace（实际上他的本质可以看成一个SSH服务器），配合vscode，我们可以模拟远程的云服务器开发

在vscode插件搜寻codespace安装，在remote栏目可以找到，要求你登陆自己的github账号
![[Pasted image 20260209112117.png]]


登陆之后你可以创建自己的一个codespace
![[Pasted image 20260209112258.png]]

![[Pasted image 20260209112327.png]]
可以选择一个仓库作为codespace
![[Pasted image 20260209112438.png]]

要你选择一个分支
>实际上这些图形化的操作和我们前面所讲的命令操作无二异，只是加上了UI界面，你也可以自己写一个（）

![[Pasted image 20260209112603.png]]

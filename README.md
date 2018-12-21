# Git Hooks、GitLab CI持续集成以及使用Jenkins实现自动化任务


## 前言

在一个共享项目（或者说多人协同开发的项目）的开发过程中，为有效确保团队成员编码风格的统一，确保部署方式的统一，等等（git的用户经常会涉及到此类场景），常常会使用类似 [Git Flow](http://flc.io/2015/12/381.html) 这种比较复杂的工作流开发模式。在较大型的项目中，虽然这种工作流模式比较成熟，但在分支处理方面，这种工作流就会造成较多的重复劳动。

因此，如果能借助某些工具来**自动化处理这些重复性事务**，比如**自动合并分支**，那么对于提升我们的工作效率，将会有很大的帮助。

本文将从以下三种方法对**自动化任务处理**做介绍，并对每一种方法的优缺点做个简单的总结，以及在实际工作中我们该如何做出选择。


#### 三种实现方法：

- 客户端与服务端 Git hooks
- GitLab-Runner 以及编写 .gitlab-ci.yml 文件
- GitLab Webhooks 与 Jenkins 的配合使用

## Git 钩子

Git hooks是基于事件的。当你执行特定的git指令时，该软件会从git仓库下的hooks目录下检查是否有相对应的脚本，如果有就执行。

有些脚本是在动作执行之前被执行的，这种“先行脚本”可用于实现代码规范的统一、完整性检查、环境搭建等功能。有些脚本则在事件之后被执行，这种“后行脚本”可用于实现代码的部署、权限错误纠正（git在这方面的功能有点欠缺）等功能。

### 安装一个钩子
钩子都被存储在Git目录下的hooks子目录中。也即绝大部分项目中的.git/hooks。当你用git init初始化一个新版本库时，Git 默认会在这个目录中放置一些示例脚本。这些脚本除了本身可以被调用外，它们还暴露了被触发时所传入的参数。这些示例的名字都是以 .sample 结尾，如果想启用它们，移除这个后缀即可。

把一个正确命名且可执行的文件放入 Git 目录下的 hooks 子目录中，即可激活该钩子脚本。这样一来，它就能 被 Git 调用。

### 客户端和服务器端 Git hooks

git hooks 采用 事件机制, 在相应的操作(比如 git commit / git merge)下触发, 分为 2 种:

 * 服务端 hooks, github 的 webhooks 就是在此基础上建立起来的；

 * 客户端 hooks, 每个 git 版本库的 .git/hooks/ 文件夹下就有可以使用的例子。

>注意: 客户端 hooks 并不会同步到版本库中

客户端钩子由诸如提交和合并这样的操作所调用，而服务器端钩子作用于诸如接收被推送的提交这样的联网操作。

>客户端钩子位于项目根目录 your\_project/.git/hooks 文件夹下
>
>服务端钩子则位于 your\_project.git 文件夹下的 hooks 和 custom_hooks

#### hooks与custom_hooks文件夹

![](https://img2018.cnblogs.com/blog/1414425/201812/1414425-20181221152123396-1023631782.png)

![](https://img2018.cnblogs.com/blog/1414425/201812/1414425-20181221152130647-151700382.png)

可以看到GitLab为我们创建了一个软连接连接到了GitLab自定义的钩子目录,这样所有创建的项目都可以使用同一个脚本规则，减少了维护成本。

那我们如何结合GitLab定制自己的脚本呢？git会首先触发GitLab的脚本，然后GitLab执行完自己的脚本文件后会再调用掉用户放在custom\_hooks下的脚本，所以我们只需要将我们定制好的脚本放在custom_hooks下即可，脚本名称和之前一样。例如：touch post-receive，这个脚本理论上可以使用任何脚本语言例如Perl、Python、Ruby等，不过执行这个脚本的用户将是git，要注意git用户对系统的操作权限，还要注意post-receive这个脚本需要能够有执行权限。

### 客户端与服务端钩子图示

![](https://img2018.cnblogs.com/blog/1414425/201812/1414425-20181221152153512-1700902063.png)

（图片来自于网络）

### 示例
#### post-receive 推送代码后自动部署
将目录切换至 ../BRIDGE_REPO.git/hooks，用 cp post-receive.sample post-receive 复制并重命名文件后用 vim post-receive 修改。其内容大致如下：

```
#!/bin/sh

unset GIT_DIR

NowPath=`pwd`
DeployPath="../../www"

cd $DeployPath
git pull origin master

cd $NowPath
exit 0
```
使用 chmod +x post-receive 改变一下权限即可，服务器端的配置就基本完成了。

## GitLab-CI与GitLab-Runner
### 持续集成（Continuous Integration）

要了解GitLab-CI与GitLab Runner，我们得先了解持续集成是什么。
>持续集成是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试)来验证，从而尽快地发现集成错误。许多团队发现这个过程可以大大减少集成的问题，让团队能够更快的开发内聚的软件。

### GitLab-CI
GitLab-CI就是一套配合GitLab使用的持续集成系统（当然，还有其它的持续集成系统，同样可以配合GitLab使用，比如接下来要说的Jenkins）。

持续集成，我们通常使用CI来做一些自动化工作，比如程序的打包，单元测试，部署等，这种构建方式避免了打包环境差异引起的错误，提高了工作效率。Gitlab-CI是Gitlab官方提供的持续集成服务，我们可以在仓库的根目录下新建.gitlab-ci.yml文件，自己定义**持续集成流程模板**，并且在Gitlab中配置runner，在之后的每次提交或合并中将会触发构建，并且可以通过Gitlab的hook, 在代码提交的各个环节自动地完成一系列的构建工作，总之对于一些非复杂性的集成需求，都是可以满足的。

实际上，GitLab-CI中有一个概念叫 **Pipeline** ，一次 Pipeline 其实相当于一次构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程。任何提交或者 Merge Request 的合并都可以触发 Pipeline。

思考：
为什么不是 GitLab CI 来运行那些构建任务？

>一般来说，构建任务都会占用很多的系统资源 (譬如编译代码)，而 GitLab CI 又是 GitLab 的一部分，如果由 GitLab CI 来运行构建任务的话，在执行构建任务的时候，GitLab 的性能会大幅下降。

>GitLab CI 最大的作用是管理各个项目的构建状态，因此，运行构建任务这种浪费资源的事情就交给 GitLab Runner 来做了！

>因为 GitLab Runner 可以安装到不同的机器上，所以在构建任务运行期间并不会影响到 GitLab 的性能~

### GitLab-Runner
GitLab-Runner是配合GitLab-CI进行使用的。一般地，GitLab里面的每一个工程都会定义一个属于这个工程的软件集成脚本，用来自动化地完成一些软件集成工作。当这个工程的仓库代码发生变动时，比如有人push了代码，GitLab就会将这个变动通知GitLab-CI。这时GitLab-CI会找出与这个工程相关联的Runner，并通知这些Runner把代码更新到本地并执行预定义好的执行脚本。

所以，GitLab-Runner就是一个用来执行软件集成脚本的东西。你可以想象一下：Runner就像一个个的工人，而GitLab-CI就是这些工人的一个管理中心，所有工人都要在GitLab-CI里面登记注册，并且表明自己是为哪个工程服务的。当相应的工程发生变化时，GitLab-CI就会通知相应的工人执行软件集成脚本。

#### Runner一共有三种类型
- 本地Runner
- 普通的服务器上的Runner
- 基于Docker的Runner

### MAC环境安装gitlab-ci-multi-runner
* sudo curl --output /usr/local/bin/gitlab-runner https://gitlab-ci-multi-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-ci-multi-runner-darwin-amd64
* gitlab-runner register
* gitlab-runner install (需到相关的项目文件夹下)
* gitlab-runner start
* gitlab-runner run

###### 具体说下gitlab-runner register
>Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/ci):
http://xxxxxx   	// 在这里输入gitlab安装的服务器ip/ci 即可

>Please enter the gitlab-ci token for this runner:
xxxxxxxxxxxxxxxxxx    // 这里的token可通过Gitlab上的项目Runners选项查看

>Please enter the gitlab-ci description for this runner:[E5]:demo       
// 这里填写一个描述信息

>Please enter the gitlab-ci tags for this runner (comma separated):
demo       // 在这里填写tag信息，多个tag可通过逗号,分割。
tag：一个项目可能有多个runner，是根据tag来区别runner的。

>Registering runner... succeeded.     runner=eaYyokc5

>Please enter the executor: docker, docker-ssh, parallels, shell, ssh, virtualbox, docker+machine, docker-ssh+machine:
shell            // 在这里需要输入runner的执行方式，直接输入shell

>Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!   // 出现这样信息表示服务端的配置就已经成功结束了。

#### 如何编写 .gitlab-ci.yml 文件
如果你在项目仓库里面加入.gitlab-ci.yml文件，同时给项目配置了gitlab-runner, 那么每次提交代码或者合并 mr , 都会触发你的 CI Pipeline （持续集成管道）。

![](https://img2018.cnblogs.com/blog/1414425/201812/1414425-20181221152214009-162997384.png)


```
stages:
  - deploy
deploy:
    stage: deploy
    script:
      - echo "start deploy....."
      - deploy
    only:
      - master
    tags:
      - shell
```
其中deploy是编写的shell脚本，可以实现将要发布的内容自动部署到发布目录下：

```
#!/bin/bash
deploy_path="xxx"
project_path="xxx;
judge_path = "$deploy_path/$project_path"
if [ ! -d "$judge_path" ]
then
   project_url="xxx.git"
   git clone $project_path $deploy_path
else
   cd $deploy_path
   git pull
fi
```

#### .gitlab-ci.yml配置详解请参考：

[gitlab ci/cd .gitlab-ci.yml配置详解](https://fennay.github.io/gitlab-ci-cn/gitlab-ci-yaml.html)

[官方GitLab文档](https://docs.gitlab.com/ce/ci/yaml/README.html)

[官方GitLab文档翻译](https://github.com/Fennay/gitlab-ci-cn)

## Gitlab Webhooks
Webhooks 允许第三方应用监听 GitLab 上的特定事件，在这些事件发生时通过 HTTP POST 方式通知( 超时5秒) 到第三方应用指定的 Web URL。 例如项目有新的内容 Push，或是 Merge Request 有更新等。 WebHooks 可方便用户实现自动部署，自动测试，自动打包，监控项目变化等。

webhooks, 可以在 pull request / merge master 等几个场景下, 设置异步回调通知(http 请求)。这个背后就是 git hooks 在起作用。

因此，利用 WebHooks 的特性，可配合 Jenkins 实现一系列的自动化任务。

## Jenkins
Jenkins是一个用Java编写的开源的持续集成工具，可以与Git打通，监听Git的merge, push事件，触发执行Jenkins的指定任务(job)。例如发布的任何一个环节都可自动完成，无需太多的人工干预，有利于减少重复过程以节省时间和工作量等。

![](https://img2018.cnblogs.com/blog/1414425/201812/1414425-20181221152230490-531786834.png)

### 实例：Jenkins、Gitlab webhooks实现开发分支自动合并

#### 步骤梳理

1. GitLab上准备一个web工程； 
2. GitLab上配置Jenkins的webhook地址；  
3. Jenkins安装GitLab Plugin插件； 
4. Jenkins配置GitLab访问权限； 
5. Jenkins上创建一个构建项目，对应的源码是步骤1中的web工程； 
6. 修改web工程的源码，并提交到GitLab上； 
7. 检查Jenkins的构建项目是否会触发自动任务脚本。

###### （Jenkins Job 和 GitLab 的关联，在网上已经有许多完善的文档了，在这里就不赘述了）
##### 以下为开发分支develop自动合并master分支的脚本示例，仅供参考：
```
#!/bin/sh
echo *****************Start*****************
date
# 获取最近一次提交的 commit id
sha1=`git rev-parse HEAD`
# 获取姓名及邮箱，来配置git提交者信息
name=`git show $sha1 | grep 'Author:' | cut -d' ' -f2`
email=`git show $sha1 | grep 'Author:' | cut -d' ' -f3 | sed -e 's/<//g' | sed -e 's/>//g'`
echo '当前提交人信息:'
echo $name 
echo $email 
git config --global user.name $name
git config --global user.email $email
echo '***************** git checkout develop & git pull:'
git checkout develop
git pull
# develop合并master
echo '***************** git merge origin/master:'
conflict=`git merge origin/master`
echo $conflict | grep 'CONFLICT'
if [ $? -ne 0 ]; then
    echo '***************** git push origin HEAD:'
    git push origin HEAD
    echo '***************** git status:'
    git status
else
    git status
    echo 'Automatic merge failed...'
    echo 'Please fix conflicts and then commit the result...'
    exit 1
fi
echo *****************End*****************

```

#### 三种实现方法的优缺点对比：

- 客户端与服务端 Git hooks ：如果仅涉及客户端钩子，用这种方法比较好，比如 husky 这个插件；但如果是服务端钩子，就必须在服务端配置才可使用，比如 post-receive 钩子；
- GitLab-Runner 以及编写 .gitlab-ci.yml 文件：需服务端安装 gitlab-runner 来支持自动化脚本的执行；
- GitLab Webhooks 与 Jenkins 的配合使用：Jenkins是比较成熟的第三方持续集成系统，可与GitLab完美的结合使用，但配置过程仍是稍显复杂，但在自动化任务处理方面，Jenkins无疑是个较好的选择。

## 参考资料

[Pro Git](http://git.oschina.net/progit/)

[GitLab 官方文档](https://docs.gitlab.com/ce/ci/yaml/README.html)

[Git Hooks](https://segmentfault.com/a/1190000000356487)

[GitLab-CI 与 GitLab-Runner](https://www.cnblogs.com/cnundefined/p/7095368.html)

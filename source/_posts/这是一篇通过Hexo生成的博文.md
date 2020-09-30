---
title: 这是一篇通过Hexo生成的博文
date: 2018-03-20 14:33:42
tags:
- 建站
---
## 背景
第一次整博客不知道是什么时候了，可能是大一，当时用Github的学生优惠包买了DigitalOcean的VPS，首先是用来科学上网的，充了5美刀用了一年也没用完，然后就想着玩玩博客，然后就照着教程用LAMP+WordPress来傻瓜式的整了一个网站，当时水平有限，没什么可以分享的东西，更多的是感受了一下搭建网站的流程，但也只是一知半解。

然后第二版的博客用的是Django框架，根据[Django博客教程](https://www.zmrenwu.com/courses/django-blog-tutorial/)来建起来的。不得不说Django框架真的很灵活并且易上手，当时跟着这个教程大概一下午左右就能把博客弄得有模有样，并且对于Django框架也有了一定了解，使得后来许多课程需要设计和实现前后端分离时Django都是我的首选后端框架。

然后就是现在，想着没必要用Django这种框架来实习博客，有点杀鸡用牛刀的感觉，不如静态页面来的轻量高效，并且有不少专门为博客准备的好看实用的框架，部署方便，主题丰富，何乐而不为呢。其中Hexo自然是首选，类似的还有Hugo, Jekyll，如果感兴趣都可以去试一下。

## Hexo使用
可以直接参考Hexo中文文档，作者是个台湾老哥，中文文档写的还是比较清晰的：[Hexo文档](https://hexo.io/zh-cn/docs/)

只需执行：
```
$ npm install -g hexo-cli
```
然后：
```
$ hexo init <folder>
$ cd <folder>
$ npm install
```
随后即刻就可以开始写作，只需：
```
$ hexo new <title>
```
即可。可以在文件夹中看到`source/_posts`目录，在里面会出现刚才新建的blog名字，就可以在里面开始编写了。如果需要在博客中放入图片，可以直接在`source/_posts`目录下新建和blog相同的名字的文件夹，把图片等资源放在文件夹下即可。

同时，我们也可以发现目录中有`_config.yml`文件，这个就是用来进行相关配置的，可以参考Hexo文档进行需要的配置，稍后讲到部署时就会涉及这个配置文件。

我使用的主题是[maupassant](https://github.com/tufu9441/maupassant-hexo)，主题也有自己的配置，可以在`theme/<theme name>/_config.yml`中配置，具体配置方法根据不同主题而异。我这边主要配置了`discus`以及博客上方menu的布局和内容。

在完成了写作后，可以在本地使用命令：
```
$ hexo server 
```
来起服务，并访问对应端口来进行预览。预览完毕，就可以进行部署了。

## 部署
最方便的部署方案是直接使用Github Pages，我也推荐直接使用Github Pages来部署，但因为我最开始并没有使用它，直接部署在了阿里云的vps上，后续也索性不再迁移，所以我这边介绍的是部署在vps上的方案。

因为hexo本身就是静态页面博客，所以想要通过域名访问到博客自然是需要使用到反向代理的，因此我们需要在vps上首先安装后Nginx，然后将hexo生成的静态资源文件放在合适的位置，并配置好Nginx，就可以成功访问了。但这样的部署流程显然是不够便捷且简单的，这也是为什么hexo提供了部署功能的原因。这边我们使用hexo提供的git部署方式。因此在本地需要先进行好git的相关配置，这里不做介绍。还有一个核心步骤是要生成ssh-key。
```
$ ssh-keygen -t rsa -C 'youremail@example.com'
```

然后我们需要在vps也进行git的安装和配置，我们首先为vps专门创建一个git用户和用户组，专门进行git操作，这也是考虑到了安全性。
```
$ useradd -m git            # 创建 git 用户，也可以用 adduser
$ passwd git                # 设置密码
$ chmod 740 /etc/sudoers    # 设置sudoers文件可写
$ vim /etc/sudoers
```
然后修改sudoers文件：
```
git   ALL=(ALL)     ALL
```
修改完成后恢复sudoers为不可写：
```
chmod 440 /etc/sudoers
```
随后，为了使之后的部署不再需要验证，我们将之前生成在`~/.ssh`目录下的`id_rsa.pub`也就是公钥文件复制到vps用户目录也就是`/home/git/.ssh/authorized_keys`文件中去，之后部署也就不再需要输入密码了。

然后我们再vps创建一个git的bare仓库，然后配置git-hook，还是在git用户也就是`/home/git`文件夹中创建：
```
$ git init blog.git --bare
```
然后进入`blog.git\hook`文件夹后，创建`post-receive`文件，这个脚本就是我们本地完成博客调用deploy更新到远程仓库后执行的脚本，在`post-receive`文件中输入：
```
#! /bin/sh
git --work-tree=/blog/www --git-dir=~/blog.git checkout -f
```
这边的`/blog/www`就是实际静态博客页面存放的地方，可以根据需求自行修改，在后面的Nginx配置时保持一致即可。然后我们修改`post-receive`文件为可执行：
```
chmod +x post-receive
```
随后我们就可以验证一下到这一步我们的配置是否正确，在本地的hexo目录中打开配置文件，可以看到`deploy`配置项，我们修改其为：
```
deploy:
  type: git
  repo: git@<your vps ip>:/your/path/blog.git
  branch: master
  message: 
```
然后运行：
```
$ hexo g && hexo d
```
我们可以去之前设置的目录下是否出现了页面文件，在这边是`/blog/www`目录。

最后，我们配置Nginx，这边不多做展开，和正常配置Nginx并无区别。如果配置成功并启动完成后，不出意外的话，已经能够通过域名访问到我们的博客了。

## 搭配CI
现在我们的workflow还是比较简单的，只需将我们的博客push至Github，然后不管身处何地，只需在电脑上clone我们的博客仓库，然后开始写作，完成后使用`hexo g && hexo d`即可将博客更新至vps。但还是存在一些问题：
1. 每台写博客的电脑上都必须安装hexo
2. 在新电脑上除了需要安装hexo之外，还需要再次配置密钥，或输入密码

因此我们可以搭配Travis-CI使用，将生成页面和部署的功能放到CI上去完成。同时，Travis-CI与Github的搭配非常不错，我们每次写作完成后，只需push到Github，会自动触发Travis-CI的workflow，然后帮我们完成生成和部署的工作。可能会有人问，CI的虚拟机如何通过vps的鉴权呢？如果用ssh-key的私钥，我们怎么存储呢？我们可以利用travis-CI的加密工具，将密文存储在Github，而解密所需的key和iv通过环境变量的方式存储在Travis-CI，这样就可以在每次部署时进行一次解密，来使得我们的部署顺利进行。

首先我们去Travis-CI处打开我们博客的CI：

![](这是一篇通过Hexo生成的博文/blog.png)

然后安装travis命令行工具
```
$ sudo apt-get install ruby-dev
$ gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/ # 改为国内gem源
$ gem install travis
```
然后登录：
```
travis login --auto
```
这边会需要我们的Github账号和密码。

然后在博客目录下，我们将之前生成的私钥`~/.ssh/id_rsa`加密：
```
$ travis encrypt-file ~/.ssh/id_rsa --add
```
不出意外的话就能在.travis.yml文件中看到形如：
```
- openssl aes-256-cbc -K ... ... 
```
这样的命令，然后我们修改为：
```
- openssl aes-256-cbc -K $encrypted_7afe5c18eac3_key -iv $encrypted_7afe5c18eac3_iv
  -in travis.key.enc -out ~/.ssh/id_rsa -d
```
切记，这边的`travis.key.enc`是加密后的私钥文件，原来的id_rsa文件一定不能暴露。

然后我们继续修改`.travis.yml`文件：
```
language: node_js
node_js:
  - 10

branches:
  only:
  - master

cache:
  directories:
  - node_modules
  - themes

before_install:
- openssl aes-256-cbc -K $encrypted_7afe5c18eac3_key -iv $encrypted_7afe5c18eac3_iv
  -in travis.key.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
- git config --global user.name "xxx"
- git config --global user.email "xxx@gmail.com"
- npm install

script:
- hexo clean
- hexo generate
- hexo deploy

addons:
  ssh_known_hosts:
  - github.com
  - crabfishhh.com
  - blog.crabfishhh.com
  - <your ip>
```
这边我给出了我的配置，可以参考一下，可根据自己实际情况进行修改。

至此，我们的配置算是结束了，可以验证一下。我们之后的workflow就会很简单，clone下来我们的博客后，进行修改或写作，完成后直接push，就会触发Travis-CI，而Travis-CI会帮我们完成生成及部署的工作，然后稍等片刻就能在网站上看到我们博客的更新啦。
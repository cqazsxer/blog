---
layout: w
title: Travis CI + frp本地服务器穿透内网 实现持续集成
date: 2018-01-15 12:00:00
tags: Jenkins nodejs
--- 

# Travis CI + frp本地服务器穿透内网 实现持续集成

标签（空格分隔）： 持续集成 TravisCI

---

[toc]

## **注册配置 Travis**

直接用github登录就行了，选择其中一个或多个你需要集成的项目，开启 build 即可。

## **添加 .travis.yml**

最简化配置文件：
```yaml
language: node_js
node_js:
- '8'
```

我的配置文件([官方doc][1])
```yaml
language: node_js
node_js:
- '8'
cache:
  directories:
    - "node_modules"
install:
- cd root
- npm i -g quasar-cli
- npm i 
script:
- quasar build
- cd ../
- sudo chmod 600 deploy.sh
- ./deploy.sh
addons:
  ssh_known_hosts: 207.148.115.242
before_install:
- openssl aes-256-cbc -K $encrypted_219f85e6348a_key -iv $encrypted_219f85e6348a_iv
  -in gaofeng_xshell_new_private.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
after_success:
- ls ~
- ssh root@207.148.115.242 StrictHostKeyChecking=no 'pwd'
```
项目根本有该文件后， 每次pushTravis会监听文件变化， 并**触发构建。**

## **使用SSH免密登录到远程服务器**

要想通过Travis登录到远程服务器， 则必须让Travis 登录到远程服务（废话）。

Travis不像交互式终端， 不能输入密码, 所有只能ssh登录！你的私钥也不能设密码！

Travis的方案是**用[EncryptingFiles][2] 加密ssh私钥来进行远程。**

SSH的登陆原理：

+ 在客户端生成一个公钥/私钥对。
+ 将公钥内容保存到服务器 ~/.ssh/authrized_keys。
+ 客户端发起连接请求时，发私钥过去。

### **安装依赖**

注： ruby需要1.9.3以上版本

```
$ gem install travis
```

装不上的话换源或者用代理：
``` 
$ gem sources -l
$ gem update --system
$ gem sources --add https://gems.ruby-china.org/
```

gem有问题报错：原因是`centos`最小化安装没有`gcc`环境，安装即可。
```
yum -y install gcc mysql-devel ruby-devel rubygems
```

### **在命令行中登录travis**
```
$ travis login

We need your GitHub login to identify you.
This information will not be sent to Travis CI, only to api.github.com.
The password will not be displayed.

Try running with --github-token or --auto if you don't want to enter your password anyway.

Username: xxx@xxx.xxx
Password for xxx@xxx.xxx: ***
Successfully logged in as demo!
```

### **加密本地密钥**


注意： `encrypt-file` **仅支持** `mac` 或者 `linux`,  window加密的在linux上解密会**报错(**[link][3])。
    
> There is a report of this function not working on a local Windows machine. Please use the WSL (Windows Subsystem for Linux) or a Linux or OS X machine.

切换到项目根目录即 `.travis.yml` 目录下。
```
# 这个是你的私钥 --add参数会把加密的私钥解密命令插入到.travis.yml，Travis解密时要用到的
$ travis encrypt-file ~/.ssh/id_rsa --add

# xxx/xxx：你的项目repo
Detected repository as xxx/xxx, is this correct? |yes| yes
encrypting ~/.ssh/id_rsa for xxx/xxx
storing result as id_rsa.enc
storing secure env variables for decryption

Make sure to add id_rsa.enc to the git repository.
Make sure not to add ~/.ssh/id_rsa to the git repository.
Commit all changes to your .travis.yml.
```
`$encrypted_d89376f3278d_key` 、`$encrypted_d89376f3278d_iv`被当作环境变量插入。

这个时候`.travis.yml`会多出几行：
```
before_install:
  - openssl aes-256-cbc -K $encrypted_d89376f3278d_key -iv $encrypted_d89376f3278d_iv
  -in id_rsa.enc -out ~\/.ssh/id_rsa -d
```
稍加修改
```
before_install:
  - openssl aes-256-cbc -K $encrypted_d89376f3278d_key -iv $encrypted_d89376f3278d_iv
    -in id_rsa.enc -out ~/.ssh/id_rsa -d
  #这里改下密钥权限
  - chmod 600 ~/.ssh/id_rsa
# fix 第一次登录远程服务器会出现 SSH 主机验证问题。
addons:
  ssh_known_hosts: your-ip
```
命令中参数解释：

+ `-in` 待解密的文件， 在Travis根目录。
+ `-out` 解密后的文件，之后ssh验证会用到。


## **执行部署脚本**
```
after_success:
  - ssh your-user@your-ip "./build.sh"
```

.sh权限问题：
比如外部系统的文件

    chmod 600 build.sh

或者

    before_install:
      - chmod +x build.sh
      
或者 [直接bash运行][4]

    bash build.sh

> Another option would be to run the script using bash, this would omit the need to modify the files' permissions.
In this case you're not executing the script itself, you're executing bash or sh which then runs the script. Therefore the script does not need to be executable.


  [1]: https://docs.travis-ci.com/
  [2]: https://docs.travis-ci.com/user/encrypting-files/
  [3]: https://docs.travis-ci.com/user/encrypting-files/#Caveat
  [4]: https://stackoverflow.com/questions/42154912/permission-denied-for-build-sh-file
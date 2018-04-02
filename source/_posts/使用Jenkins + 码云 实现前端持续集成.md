---
layout: w
title: 使用Jenkins + 码云 实现前端持续集成
date: 2018-01-10 15:29:07
tags: Jenkins nodejs
---

# 使用Jenkins + 码云 实现前端持续集成

标签（空格分隔）： Jenkins 持续集成

[toc]

---

## **1. 安装**
`JENKINS_HOME` 根目录默认设置在 `~/.jenkins`。（win: `C:\Users\gf\.jenkins`)
建议使用war包进行安装[下载][1]（需求`jdk` > 8环境）

### **1.1 war包**
在指定端口开启服务
```
java -jar jenkins.war --httpPort=8008
```
### **1.2 exe文件安装**
安装完会在8080启动服务

可在 `Jenkins/jenkins.xml` 中`arguments`设置httpPort字段默认端口。

```
// JENKINS_HOME 目录下
net start jenkins // 开启服务
net end jenkins // 关闭
```
## **2. 相关插件安装**
安装推荐的插件即可。
其他重要的插件：

+  NodeJS Plugin
+  Generic Webhook Trigger

## **3. 配置环境变量**

![image_1c3k2nj501lop1dbu1sduj001qk1m.png-29.6kB][2]



## **4. webhook配置**
需求插件： `Generic Webhook Trigger` 

### 4.1 码云端配置：
[相关文档][3]

在“系统管理–管理用户–用户列表–admin处点击进去–左边侧边栏–设置”设置用户`API Token`、`User ID`

URL格式:

    http://<User ID>:<API Token>@<Jenkins IP地址>:端口/generic-webhook-trigger/invoke 
    http://有读权限的用户名:该用户名Token@jenkis地址/generic-webhook-trigger/invoke

参考：

    http://gf:36c780ecc06beefa242b9cd9083c0260@118.178.94.163:8001/generic-webhook-trigger/invoke

测试成功的返回:

![image_1c3hqf8pa10tc1s7s1nnepnu1sjo9.png-25.5kB][4]


### 4.2 Jenkins配置：
 [jsonPath文档][5]

从码云的返回筛选字段

![image_1c5kfmnkf158d1b311otav31rru9.png-31.8kB][6]
![image_1c5kfpcueb635e1apv1l8dqv9m.png-9kB][7]

过滤项目
![image_1c5kfr1bm1gi61jc71df1at717eb13.png-14.9kB][8]


  [1]: https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins
  [2]: http://static.zybuluo.com/gao1994/kgvedncieq5caeq7nzf4uzu4/image_1c3k2nj501lop1dbu1sduj001qk1m.png
  [3]: http://git.mydoc.io/?t=154711
  [4]: http://static.zybuluo.com/gao1994/y53htlc21m5usuoczl8sveor/image_1c3hqf8pa10tc1s7s1nnepnu1sjo9.png
  [5]: https://github.com/json-path/JsonPath
  [6]: http://static.zybuluo.com/gao1994/b69sk952t7q4bmembbyus78g/image_1c5kfmnkf158d1b311otav31rru9.png
  [7]: http://static.zybuluo.com/gao1994/mjahcx9okyjuh5ibarbxmph2/image_1c5kfpcueb635e1apv1l8dqv9m.png
  [8]: http://static.zybuluo.com/gao1994/8f891lndfw51ikd4xoto6741/image_1c5kfr1bm1gi61jc71df1at717eb13.png
  
## **5. 构建步骤**

todo

## **6. 邮件提醒配置**

todo 
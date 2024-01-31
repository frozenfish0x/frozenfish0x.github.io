---
layout: 问题与解决
title: 解决Sourcetree推github不停弹登录窗Logon failed, use ctrl+c to cancel basi
date: 2023-11-27 07:49:11
tags:
---
最近有一段时间没用 GitHub，新建了一个仓库，打开 Sourcetree → + → Clone → 填入git地址，顺利拉取了仓库分支。分支一个 → 提交 → 推送，忽然弹出了GitHub登录窗口，记得以前曾经弹过一次，忘记什么情况了，那就填写用户名、密码，login吧：



红色闪眼啊，报错啊！~
git -c diff.mnemonicprefix=false -c core.quotepath=false --no-optional-locks push -v --tags origin main:main
Logon failed, use ctrl+c to cancel basic credential prompt.


<!--more-->
“Logon failed...” 大概是手快，密码输错了吧，再来一遍，什么？还是失败，估计是密码自己也记错了。赶紧上官网登录试一下，实在不行重置。。。咦？还是刚才的用户名、密码，官网顺利登录，没问题啊。回来 Sourcetree 再试一遍，错误照常。看来八成 Sourcetree 出毛病了。baidu 吧，哦，有人说到什么 Git 软件更新了，什么登录机制变了，不能再用登录框的方式登录了，更新后就不会出现登陆框了，会自动连接到 GitHub 官网验证。。。云云。嗯，和我的情况符合，赶紧点击文章中给的官方 Git 链接，下载了最新版本的 Git，安装。保险起见重启电脑，打开 Sourcetree，提交，哇，居然还是有弹窗啊，稳定情绪，说不定这次登录就可以了，输入用户名、密码 → login，报错依旧。看来这方法不行。继续baidu，又试了在仓库设置中修改 git 地址为 userName@github.com/xxx/xx.git 的格式，还是不行。是最后这个提示帮助了我，有网友说需要“使用OAuth进行身份验证”，翻了翻 Sourcetree 菜单，最后终于在 工具 → 选项 中找到答案。选项 → 验证 选项卡。看到现在的 gitHub 登录验证为 Basic ， 直接编辑用户，输入密码，好像无反应。新建一个吧，右上角“添加”，服务商: GitHub, 验证一定选择 OAuth，这时，下面的用户名会是禁用状态：

<!--more-->

直接点击下面的按钮“刷新 OAuth 令牌”，奇迹出现了，自动跳转到浏览器打开了 GitHub 登录界面：



赶紧登录吧，登录后：



看到授权成功！再次返回 Sourcetree 界面，看到登录窗上已经显示绿色√ ，认证成功！



确定，这时可以看到验证窗口中多了一个账户，下面 “Git 已存密码”中也添加了一个Git用户，但是两个的验证方式是不同的，这点不太明白具体意思，有时间查一下。好了，接下来，赶紧推啊。祝你也 顺 利 推 送！



方案二也可以在GitHub个人用户里生成一个新的note和tokens，第一次输入自己的GitHub账号和密码，然后在第二次输入usernuserame和password的时候输入对应的note和tokens就好了。具体操作步骤可以参考博客https://blog.csdn.net/weixin_46622106/article/details/111914231
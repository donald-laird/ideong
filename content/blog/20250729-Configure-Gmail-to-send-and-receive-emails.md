---
title: "配置 gmail 代收发邮件实现企业邮箱功能"
date: 2025-07-29T15:40:08+08:00
description: "通过配置 Gmail，实现让自己的域名实现企业邮箱收发邮件的功能"
cascade:
   type: blog
tags: 
   - gmail
   - Cloudflare 
   - 邮件服务
---

## 域名服务器的配置

第 1 步：在 Cloudflare 需要配置邮箱的域名中，打开左边栏 `Email Routing` 菜单项，点击开始 `Get Started`。

![](/images/2025/config_email_01.png)

第 2 步：配置想要命名的邮箱地址。同时配置收件时转发到你自己可以收信的邮箱地址。点击创建和下一步。

![](/images/2025/config_email_02.png)

第 3 步：等待状态为已验证，然后下一步。

![](/images/2025/config_email_03.png)

第 4 步：此时它会告诉你解释记录还未添加，点击添加记录并生效。

![](/images/2025/config_email_04.png)

第 5 步：成功后，可以在 `Routing rules` 标签页查看邮箱解析状态。此时状态为有效 `Active`。

![](/images/2025/config_email_05.png)


## 在 Gmail 中配置

第 1 步：登录 Gmail 邮箱，在右上角的设置那里打开所有设置。

![](/images/2025/config_email_06.png)

第 2 步：选择账号与导入。找到 `Send mail as` 区域，添加另一个邮箱地址。

![](/images/2025/config_email_07.png)

第 3 步：将需要导入的邮箱地址和名字输入。这里的名字作为收件人看到邮件时显示的名字。

注意：这里的 `Treat as an alias` 可以不勾选，这样收件人收到此邮箱代发的邮件时，不会显示由谁代发这样的字眼，当然这远远不够，还需要支持域名设置 DKIM、SPF 等信息。

> **为什么会显示代发字样**
> 要回答这个问题，首先要弄懂两个概念，一是信封地址，二是 sender。
> 信封地址：发件人声称的地址，也就是邮件的 from， 但这个地址不一定真实的发件地址，因为发件人可能会作假。
> sender 地址：邮件发出的真实地址。
> 信封地址（from 地址）和 sender 地址不相同，所以会显示 A 由B 代发；显示”代发”还有一个特殊含义，就是防止网络钓鱼邮件，因为信封地址非真实地址，ISP（互联网服务提供商）如果不显示代发，收件人可能误以为是从A 发出的，为了提供更多发件人信息，以便收件人甄别，所以同时显示发件人真实地址。

![](/images/2025/config_email_08.png)

打开 `Specify a different "reply-to" address`，填入同样的邮件地址，这样代表回复的时候也是用这个地址。

![](/images/2025/config_email_09.png)

第 4 步：点开下一步的前提是 Gmail 的 google 账户需要打开二级验证，同时需要生成一个 App Passwords 密码。如果 google 账户已经打开二级验证设置，那么通过这个链接【[https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)】设置一个 App Passwords，密码名称可以自行命名。

这个密码先复制出来保存好，因为关掉后就看不到了。

![](/images/2025/config_email_10.png)

第 5 步：在刚才点开下一步的时候，就出现下面设置。 

- 将 `SMTP Server` 设置为：`smpt.gmail.com`，端口保持不变
- `Username` 设置为代发送的 Gmail 邮箱地址
- `Password` 填入刚才生成的 App Passwords，注意密码中有空格不要删除。

进入下一步。

![](/images/2025/config_email_11.png)

第 6 步：这一步进行邮箱确认，请返回到 Gmail 邮箱，进行确认，如下图所示。

![](/images/2025/config_email_12.png)

第 7 步：回到账号与导入。找到 `Send mail as` 区域，将 `When replying to a message` 设置为 `Reply from the same address the message was sent to` 表示用收到邮件的地址回复回去。

![](/images/2025/config_email_13.png)

这样，通过 Gmail 代收发邮件的功能已经完成。

下面我们还可以通过 Gmail 使用第三方邮件服务来完成转发邮件操作。

## 使用 Brevo 的免费发信服务

Brevo 的免费额度一天有 300 封邮件，额度也比较大的。而且方便后续客户管理工作。

第 1 步：到 [Brevo 官网](https://brevo.com)免费注册个账号。

第 2 步：登录进去后，在右上角打开菜单，找到 `Senders, Domains & Dedicated IPs`
打开 `Domains` 标签页，添加域名。按提示对 Cloudflare 账户进行授权，它会自动为相应的域名添加解析记录。
![](/images/2025/config_email_14.png)

绑定域名成功后，它会显示如下，你可以通过 `Edit` 配置更详细的说明。

![](/images/2025/config_email_17.png)

第 3 步：在 Cloudflare 的域名解析中，找到值为 `v=spf1 ...` 的 TXT 解析记录，在原先记录中加入 `include:sendinblue.com`，它是 Brevo 的 SPF 授权域名。

![](/images/2025/config_email_15.png)

第 4 步：重新打开 Brevo 右上角的菜单，找到 `SMTP & API` 打开，它就显示出了后续可以添加到 Gmail 的 SMTP 服务器信息，包括端口号，用户名和密码。密码需要在下面点击生成一个新的密码，同样需要保存到一个地方以后可以找到，它显示一次后面就不再显示了。

![](/images/2025/config_email_16.png)

第 5 步：回到前面 *在 Gmail 中配置* 的第 5 步再次进行配置即可。

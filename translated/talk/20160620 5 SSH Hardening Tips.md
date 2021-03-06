五条强化 SSH 安全的建议
======================

![](https://www.linux.com/sites/lcom/files/styles/rendered_file/public/binary-1188510_1920_0.jpg?itok=ocPCL_9G)
>采用这些简单的建议，使你的 OpenSSH 会话更加安全。

> 创造性的零共有

当你查看你的 SSH 服务日志，很有可能都充斥着一些不怀好意的尝试性登录。这里有5条一般意义上的建议（和一些个别特殊策略），让你的 OpenSSH 会话更加安全。

### 1. 强化密码登录

密码登录很方便，因为你可以从任何地方的任何机器上登录。但是他们在暴力攻击面前也是脆弱的。尝试以下策略来强化你的密码登录。

- 使用一个密码生成工具，例如 pwgen。pwgen 会有几个选项；最有用的就是密码长度的选项（例如，`pwgen 12` 产生一个12位字符的密码）

- 不要重复使用一个密码。忽略所有有关不要记下你的密码的建议，然后关于你所有登录信息都记在一本本子上。如果你不相信我的建议，那去相信安全权威 [Bruce Schneier][1]。如果你足够仔细，没有人能够发现你的笔记本，那么这样能够不受到网络上的那些攻击。

- 你可以为你的登录记事本增加一些额外的保护措施，例如用字符替换或者增加新的字符来掩盖笔记本上的登录密码。使用一个简单而且好记的规则，比如说给你的密码增加两个额外的随机字符，或者使用单个简单的字符替换，例如 `#` 替换成 `*`。

- 为你的 SSH 服务开启一个不是默认的监听端口。是的，这是很老套的建议，但是它确实很有效。检查你的登录；很有可能22端口是被普遍攻击的端口，其他端口则很少被攻击。

- 使用 [Fail2ban][2] 来动态保护你的服务器，是服务器免于被暴力攻击

- 使用不寻常的用户名。不能让root可以远程登录，并确避免用户名为“admin”。

### 2. 解决 `Too Many Authentication Failures` 报错

当我的 ssh 登录失败，并显示「Too many authentication failures for carla」的报错信息时，我很难过。我知道我应该不介意，但是这报错确实很碍眼。而且，正如我聪慧的奶奶曾经说过，伤痛之感并不能解决问题。解决办法就是在你的 `~/.ssh/config` 文件设置强制密码登录。如果这个文件不存在，首先创个 `~/.ssh/` 目录。

```
$ mkdir ~/.ssh
$ chmod 700 ~/.ssh
```

然后在一个文本编辑器创建 `~/.ssh/confg` 文件，输入以下行，使用你自己的远程域名。

```
HostName remote.site.com
PubkeyAuthentication=no
```

### 3. 使用公钥认证

公钥认证比密码登录安全多了，因为它不受暴力密码攻击的影响，但是并不方便因为它依赖于 RSA 密钥对。一开始，你创建一对公钥和私钥。下一步，私钥放于你的客户端电脑，并且复制公钥到你想登录的远程服务器。你只能从拥有私钥的电脑登录才能登录到远程服务器。你的私钥就和你的家门钥匙一样敏感；任何人获取到了私钥就可以获取你的账号。你可以给你的私钥加上密码来增加一些强化保护规则。
使用 RSA 密钥对管理多种多样的用户是一种好的方法。当一个用户离开了，只要从服务器删了他的公钥就能取消他的登录。

以下例子创建一个新的 3072 bits 长度的密钥对，它比默认的 2048 bits 更安全，而且为它起一个独一无二的名字，这样你就可以知道它属于哪个服务器。

```
$ ssh-keygen -t rsa -b 3072 -f id_mailserver
```

以下创建两个新的密钥, `id_mailserver` 和 `id_mailserver.pub`，`id_mailserver` 是你的私钥--不传播它！现在用 `ssh-copy-id` 命令安全地复制你的公钥到你的远程服务器。你必须确保有运作中的 SSH 登录到远程服务器。

```
$ ssh-copy-id -i  id_rsa.pub user@remoteserver

/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
user@remoteserver's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'user@remoteserver'"
and check to make sure that only the key(s) you wanted were added.
```

`ssh-copy-id` 确保你不会无意间复制你的私钥。复制你的命令输出样式，带上单引号，这样测试你的新的密钥登录。

```
$ ssh 'user@remoteserver'
```

必须用你的新密钥登录，如果你为你的私钥设置了密码，它会提示你。

### 4. 取消密码登录

一旦你已经测试并且验证了你的公钥登录，取消密码登录，这样你的远程服务器就不会被暴力密码攻击。如下设置你的远程服务器的 `/etc/sshd_config` 文件。

```
PasswordAuthentication no
```

然后重启 SSH 守护进程。

### 5. 设置别名 -- 这很快捷而且很有 B 格

你可以为你的远程登录设置常用的别名，来替代登录时输入的命令，例如 `ssh -u username -p 2222 remote.site.with.long-name`。你可以使用 `ssh remote1`。你的 `~/.ssh/config` 文件可以参照如下设置

```
Host remote1
HostName remote.site.with.long-name
Port 2222
User username
PubkeyAuthentication no
```

如果你正在使用公钥登录，可以参照这个：

```
Host remote1
HostName remote.site.with.long-name
Port 2222
User username
IdentityFile  ~/.ssh/id_remoteserver
```

[OpenSSH 文档][3] 很长而且详细，但是当你掌握了基础的 SSH 使用规则之后，你会发现它非常的有用而且包含很多可以通过 OpenSSH 来实现的炫酷效果。



--------------------------------------------------------------------------------

via: https://www.linux.com/learn/5-ssh-hardening-tips

作者：[CARLA SCHRODER][a]
译者：[maywanting](https://github.com/maywanting)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创翻译，[Linux中国](https://linux.cn/) 荣誉推出

[a]: https://www.linux.com/users/cschroder
[1]: https://www.schneier.com/blog/archives/2005/06/write_down_your.html
[2]: http://www.fail2ban.org/wiki/index.php/Main_Page
[3]: http://www.openssh.com/

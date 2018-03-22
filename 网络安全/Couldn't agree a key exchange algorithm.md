#Couldn't agree a key exchange algorithm

Couldn't agree a key exchange algorithm (available: curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521)

之前一直在用Tortoisegit,突然前几天向Github提交代码的时候抛了一行这个错误，提交失败，查了好久大致意思就是加密方式太老了。

<b>原因：Putty 太老了</b>

你可以看到再提交的时候右下角有一个Putty的客户端，里面是你配置的PPK文件，就是这个东西版本太旧了。

解決方案：

<b>1 下載新的Putty</b>

我是通过卸载旧版本得Tortoisegit，并重新安装了一个新得版本，自带得更新了Putty

送一个TortoiseGit-2.6.0.0-64bit 和 Git_v2.16.0.2 需要的朋友请自行选择下载！

<b>2 使用SSH的方式提交代码</b>

可以使用SSH的方式提交代码，说白了就是你可以试试用git Bash 命令得方式提交。

不明白什么是SSH的同学自己去学习吧，送一个传送门 [在git与tortoisegit中使用openSSH与PuTTY](http://blog.csdn.net/wzx19840423/article/details/51258348)


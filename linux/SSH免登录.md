# SSH免登录

标签（空格分隔）： linux 工具

---

用ssh-keygen创建公钥（如果之前已经生成过了，忽略这一步）
```shell
$ ssh-keygen -t rsa -P '' 
# -P表示密码，-P '' 就表示空密码，也可以不用-P,这样要敲3次回车，-P就一次
```
公钥生成后，可以在``~/.ssh``下查看``id_rsa``是生成的私钥（我是不会告诉你私钥是不能轻易告诉别人的），``id_rsa.pub``是生成的公钥，

公钥生成好后，要把公钥拷贝到远程服务器上
```
# 这里以sandbox4为例
# 先把本地生成的公钥上传到sandbox4上,lhs_id.pub是自己取得名字
$ scp ~/.ssh/id_rsa.pub isuwang@*.*.*.*:lhs_id.pub
```

此时需要先手动ssh到sandbox4上（老规矩，先 ssh isuwang@\*.\*.\*.\*,然后输入密码）,在~目录下，可以看到刚才上传的公钥文件lhs_id.pub
接着进入~/.ssh目录，目录下会有一个authorized_keys的文件

```
如果你是第一个吃螃蟹的人，可能目录下还没有这个文件，那需要创建该文件，并设置访问权限：
$ touch authorized_keys
$ chmod 600 authorized_keys

```
然后把``lhs_id.pub``的内容 *追加* 到 ``authorized_keys``。注意，最好先备份一下``authorized_keys``，万一小手一抖操作错了，把别人上传的公钥也搞没了的话，可不是人人都承担得起后果。备份：``$ cp -f authorized_keys ../authorized_keys_bk``

```
cat ~/lhs_id.pub >> authorized_keys
```
至此，接下来就是见证奇迹的时刻了。回到你本地的终端，``$ ssh isuwang@*.*.*.*``，然后回车，然后就没有然后了......
如果还是觉得要记住这个该死的ip还是有些艰难，可以设置一个别名
```
$ alias sshsandbox4='ssh isuwang@*.*.*.*'
```


从此以后，远程到sandbox4就只需要：
```
$ sshsandbox4
```
但是！！！设置的别名仅在当前终端有效。当然，可以设置永久别名，不同平台方法略有不用，就自行百度吧。

更高级的方法：
```shell
# 在~/.ssh目录下新建一个名为config的文件（如果不存在的话）
$ vi config

```
```
# 然后插入以下内容
Host sandbox4
    User isuwang      # 登录到sandbox4的用户名
    HostName *.*.*.*  # 这里写sandbox4的ip或域名

Host weihu
    User weihu
    HostName *.*.*.*  # 写维护机的ip或域名
    
# 如有更多，自行继续按照以上格式写入... #号以及后面的内容保存前请先删掉

```

保存并退出vi，然后要远程到sandbox4：
```
$ ssh sandbox4
```
要远程到weihu机：
```
$ ssh weihu
```
~~妈妈再也不用担心我找不到sandbox的ip和密码了！

# SSH穿越跳板机
你若少输一次密码，便是晴天，更何况是两次。
>这里以sandbox1为例。要远程到sandbox1，首先你得先ssh到跳板机，然后在从跳板机ssh到sandbox1。

事到如今，我相信你本地机器已经生成有ssh的公钥和秘钥。
参照SSH免登录做法，你需要先把公钥scp到weihu机（快塑网唯一指定跳板机）上，由于你需要直接远程到sandbox1上，所以你的公钥同样需要拷贝到sandbox1中。但是，在本地机器是不能直接连上sandbox1的，那怎么办呢？聪明的人估计已经想到办法了：老子的公钥不已经拷贝到跳板机上吗？*__在跳板机上scp到sandbox1__* 不就得了？

当时就是这样，weihu机和sandbox1上已经有你的公钥。别忘了把你的公钥追加到对应机器的``~/.ssh/authorized_keys``中。

接着，回到本地终端。
还记得``~/.ssh``下的``config``吗? 没错，是它，是它，就是它。
```
$ vi config
```
加入以下配置：HostName按需自行改一改
```shell
# weihu 如果之前已经有weihu机的配置，这里就不需要重复配置了
Host weihu
        User weihu
        HostName *.*.*.*  # weihu机的ip或域名

# 配置sandbox1        
Host sandbox1
    User isuwang
    HostName *.*.*.*   # 此处应是sandbox1的ip或域名
    ProxyCommand ssh weihu nc %h %p 2> /dev/null  
```

保存退出。
然后``$ ssh sandbox1``试试。



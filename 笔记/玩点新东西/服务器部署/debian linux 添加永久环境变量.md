## 写在前面的话
搜索linux添加环境变量，网上已经有很多的教程，本来就几个命令还是把我搞的好惨，几个坑大牛们不指出来，我这小白就卡在里面了，写下血泪史供参考
## 关于环境变量
- **export** 命令显示当前系统定义的所有环境变量，比如 HOME，PATH等
- **echo $PATH** 命令输出环境变量中名为PATH的变量的值
- 终端下使用 export 命令只对当前用户有效，立即生效，当前终端窗口关闭后无效，想永久添加就要编辑 .bashrc 文件
- **export PATH=$PATH:用户地址1:用户地址2** 说明:一定不能少了$PATH把原有的变量引用进来，否则会覆盖
## debain

首次安装使用 Debian 的时候，很多工具都是不全的，甚至最基本的 sudo 命令都没有，会进环境变量的坑也是因为要添加这个 sudo 的命令，所以
#### su
直接使用 **su** 命令，输入root密码，直接强制调用 root账户，切记在此账户下要谨慎操作，有些是无法回退的。

```shell
su
```

#### cd

然后就是切换到root账户的家目录里去找到我们要修改的文件 .bashrc

```shell
cd ~
```

#### 编辑 .bashrc 文件

使用 gedit 或者直接使用 vi 进行编辑，gedit是可视化编辑，适合我这种小白

```shell
vi .bashrc
```

打开文件后，在文件最下面输入语句，$PATH: 后面的就是你要添加的环境变量，此处有大坑，切记，网上看到的 = 号两边是有空格的，我直接复制来用，执行到后面，就报 PATH 变更错误，一定一定一定不能有空格

```shell
export PATH=$PATH:/usr/sbin
```

#### 执行生效

以上保存退出后，执行以下语句生效并查看效果

```shell
source ~/.bashrc
echo $PATH
```

## PS

![](https://i-blog.csdnimg.cn/blog_migrate/abc62125167b810ff6233580171681a9.png)  
再说一个小坑，那就是这些命令是区分大小写的，我这小白不知道，结果瞎折腾了半天
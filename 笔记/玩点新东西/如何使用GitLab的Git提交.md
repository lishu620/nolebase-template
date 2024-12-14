# 安装Git
# 创建RSA密钥

## 配置Git用户和邮箱
桌面鼠标右键，点击Git bash here进入命令行
![[Pasted image 20241214151416.png]]
```bash
git config --global user.name "用户名"
git config --global user.email  "邮箱"
```
## 生成RSA密钥
```bash
ssh-keygen -t rsa -C "自己的邮箱"
```

> [!NOTE] 提示
> 执行密钥生成命令，基本上是一路回车既可以了，但是需要注意的是：
> 1.执行命令的过程中会提示
> 2.输入密钥的密码的（输入两次相同的，即是又一次确认密码），不需要密码直接回车就行

示例：
```bash
Generating public/private rsa key pair. #正在生成公钥/私钥对。
Enter file in which to save the key (/c/Users/mlish/.ssh/id_rsa): #输入key的保存位置，直接回车即可。
Enter passphrase (empty for no passphrase): #如果不想设置密码直接不输入内容，按 Enter键即可。
```

> [!WARNING] 注意
> SSH文件存放在C:/User/用户/.ssh下，id_rsa为私钥，id_rsa.pub为公钥。
## 上传RSA公钥
打开id_rsa.pub文件，复制里面的内容
登录GitLab账号，点击头像，Setting->SSH Keys,将复制的公钥粘贴到Key文本框中，title随便添加就可以了。
Git/GitLab连接配置完成，接下来就可以clonegitlab上的项目了。
git公钥链接gitlab或gitlab形式差不多类似。
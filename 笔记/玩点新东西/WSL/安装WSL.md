---
tags:
  - WSL
---
## 安装WSL命令

现在，可以使用单个命令安装运行 WSL 所需的一切内容。 在管理员模式下打开 PowerShell 或 Windows 命令提示符，方法是右键单击并选择“以管理员身份运行”，输入 wsl --install 命令，然后重启计算机。 ^a063a0

```powershell
wsl --install
```

^c0a6d2

此命令将启用运行 WSL 并安装 Linux 的 Ubuntu 发行版所需的功能。 （[可以更改此默认发行版](https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands#install)）。 ^1390e8

如果你运行的是旧版，或只是不想使用 install 命令并希望获得分步指引，请参阅[旧版 WSL 手动安装步骤](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual)。 ^cf96b9

首次启动新安装的 Linux 发行版时，将打开一个控制台窗口，要求你等待将文件解压缩并存储到计算机上。 未来的所有启动时间应不到一秒。 ^dcc732

> [!warning] 警告
> 仅当根本没有安装 WSL 时，上述命令才有效。 如果运行 `wsl --install` 并查看 WSL 帮助文本，请尝试运行 `wsl --list --online` 以查看可用发行版列表并运行 `wsl --install -d <DistroName>` 以安装一个发行版。 若要卸载 WSL，请参阅[卸载旧版 WSL](https://learn.microsoft.com/zh-cn/windows/wsl/troubleshooting#uninstall-legacy-version-of-wsl) 或[注销或卸载 Linux 发行版](https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands#unregister-or-uninstall-a-linux-distribution)。

^dad44d


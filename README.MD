# wsl2试验安装记录

2004 may update 更新了wsl2，尝试安装

## 安装wsl2

先更新2004 may update

然后以管理员身份打开 PowerShell 并运行,以安装wsl和开启wsl2所需的“虚拟机平台”：

```shell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

将 WSL 2 设置为默认版本

```shell
wsl --set-default-version 2
```

## 安装wsl发行版

可以选择ubuntu等，但是这里选择消耗最低的alpine

安装地址为：https://www.microsoft.com/store/apps/9p804crf0395

在windows环境下管理员cmd中执行下面命令,设置alpine子系统默认登录用户为root并重启lxsmanager服务使其生效：

```shell
alpine config --default-user root
net stop lxssmanager
net start lxssmanager
```

## 安装docker

替换软件源为阿里镜像，提升安装速度

```shell
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
```

安装docker

```shell
apk add --no-cache docker openrc lxc musl-dev
```

开启远程访问。编辑`/etc/init.d/docker`文件，`command_args`所在行改成下面内容

```text
command_args="-H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 -p \"${pidfile}\" ${DOCKER_OPTS}"
```

设置docker镜像源，以加速docker镜像pull速度。创建`/etc/docker/daemon.json`文件，并填充下面内容：

```shell
{
  "registry-mirrors": ["https://pee6w651.mirror.aliyuncs.com"]
}
```

创建启动脚本。在alpine子系统里创建启动脚本文件`/etc/init.wsl`

```shell
#! /bin/sh
# docker
openrc
mkdir -p /run/openrc/
touch /run/openrc/softlevel
service docker start
# others
```

给启动脚本文件赋执行权限，并运行启动

```shell
chmod +x /etc/init.wsl
bash /etc/init.wsl
```

在Windows下 输入快捷键 WIN+R 打开运行窗口
输入 shell:startup
打开Windows 启动脚本目录
在该目录下新建 alpine-start.vbs

```vbs
Set ws = WScript.CreateObject("WScript.Shell")
ws.run "wsl -d Alpine -u root /etc/init.wsl"
```

## reference

https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#update-to-wsl-2

https://blog.csdn.net/qq_37476710/article/details/105941217

https://blog.csdn.net/hiqiming/article/details/105244147

https://cr.console.aliyun.com/cn-shanghai/instances/mirrors

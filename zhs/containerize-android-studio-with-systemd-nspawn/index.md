---
lang: zh-Hans
---

# 使用 systemd-nspawn 容器化 Android Studio

## 关于本文

- 发布于 2021 年 7 月 7 日
- [源码][source]
- [网页][page]以及[英文版][page_en]

[source]: https://github.com/liolok/liolok.com/blob/master/zhs/containerize-android-studio-with-systemd-nspawn/index.md
[page_en]: https://liolok.com/containerize-android-studio-with-systemd-nspawn/
[page]: https://liolok.com/zhs/containerize-android-studio-with-systemd-nspawn/

## 创建 Btrfs 子卷

```console
# su # 切换到 root 用户
# cd /var/lib/machines
# container_name=android-studio
# btrfs subvolume create $container_name
```

然后就可以方便地对容器进行[快照][snapshot]以及[迁移][migrate]了。

[snapshot]: https://wiki.archlinux.org/title/Btrfs#Snapshots
[migrate]: https://wiki.archlinux.org/title/Btrfs#Send/receive

## 安装最小化的 Ubuntu

> 参考：[systemd-nspawn - ArchWiki](https://wiki.archlinux.org/title/Systemd-nspawn#Create_a_Debian_or_Ubuntu_environment)

当前最新的长期支持版 Ubuntu 代号为 `focal`。

```console
# codename=focal
# repository_url='https://mirrors.bfsu.edu.cn/ubuntu/'
# debootstrap --include=systemd-container \
--components=main,universe $codename $container_name $repository_url
```

安装完成后，运行 `systemd-nspawn --machine=$container_name` 即可启动进入崭新的容器。

## 网络

可能需要加上 `--bind-ro=/etc/resolv.conf` 参数才能让容器里的 DNS 正常工作。

想要检查网络状态的话，可以使用命令 `ping 1.1.1.1` 和 `ping archlinux.org`：如果前者通后者不通，说明是 DNS 问题。

## 32 位库

在容器中[启用 32 位架构][multiarch]并[安装库][libs]。

[multiarch]: https://wiki.debian.org/Multiarch/Implementation#Using_multiarch
[libs]: https://developer.android.com/studio/install#64bit-libs

```console
# dpkg --add-architecture i386
# apt-get update
# apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386 lib32z1 libbz2-1.0:i386
```

## Android Studio

可以在容器中使用 `curl`/`wget` 下载[最新版本的包][download]，也可以在主机下载之后挂在到容器：

[download]: https://developer.android.com/studio#downloads

```console
# host_as_archive='/data/Downloads/android-studio-ide-202.7486908-linux.tar.gz'
# systemd-nspawn --machine=$container_name --bind-ro=$host_as_archive:/tmp/as.tar.gz
```

在容器中添加一个专门的普通用户，就叫「android」吧，然后把 Android Studio 解压到用户目录：

```console
# useradd --create-home android
# su - android
# tar --extract --verbose --file=/tmp/as.tar.gz
# mv android-studio studio
```

现在 Android Studio 已经安装到了 `/home/android/studio/`，启动脚本在
`/home/android/studio/bin/studio.sh`：

```console
# user=android
# startup_script='/home/android/studio/bin/studio.sh'
# systemd-nspawn --machine=$container_name --user=$user $startup_script
```

哎呀，这启动不了，我忘了 [Xorg 相关处理][xorg]了。命令行参数太多，从这一步开始，想要在图形界面运行 Android Studio 就要着手撰写脚本了。

[xorg]: https://liolok.com/run-desktop-app-with-systemd-nspawn-container/#xorg "Run Desktop App with systemd-nspawn Container"

## 修复依赖

> 太长不看：`apt-get install libxext6 libxrender1 libxtst6 libxi6 libfreetype6 fontconfig`

这份[日志](../../containerize-android-studio-with-systemd-nspawn/missing-lib.log)关键是缺少库文件 `libXext.so.6`，那我们来试试「反向查询」：

```console
# systemd-nspawn --machine=$container_name --bind-ro=/etc/resolv.conf
# apt-get install apt-file
# apt-file update
# apt-file search libXext.so.6
```

结果表明库文件对应的软件包应该是 `libxext6`。安装之后，同样的四个错误接踵而至，因此所有缺少的库如下：

```console
apt-get install libxext6 libxrender1 libxtst6 libxi6 libfreetype6
```


Android Studio 终于启动了，然后马上又崩溃了。[日志](../../containerize-android-studio-with-systemd-nspawn/font.log)多次提到「字体」，于是尝试安装了 `fontconfig` 包，这次终于正常启动了。

## 修复光标主题

将主机的图标主题挂载到容器并设置环境变量：

```console
# host_data=/usr/share
# data=/home/$user/.local/share
# systemd-nspawn --machine=$container_name \
--bind-ro=$host_data/icons:$data/icons --setenv=XCURSOR_PATH=$data/icons
```

在容器中安装 X 光标管理库：

```console
# systemd-nspawn --machine=$container_name --bind-ro=/etc/resolv.conf apt-get install libxcursor1
```

## 包装一下

我写了个 [fish 脚本][script]，还有个参考 [AUR][aur-ref] 的[桌面文件][desktop-entry]。

[script]: https://github.com/liolok/dotfiles/blob/master/.local/bin/android-studio
[desktop-entry]: https://github.com/liolok/dotfiles/blob/master/.local/share/applications/android-studio.desktop
[aur-ref]: https://aur.archlinux.org/cgit/aur.git/tree/android-studio.desktop?h=android-studio

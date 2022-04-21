# Containerize Android Studio with systemd-nspawn

- Published: 2021-07-07
- [Markdown][raw]
- [Simplified Chinese][zhs]

[raw]: https://raw.githubusercontent.com/liolok/liolok.com/master/containerize-android-studio-with-systemd-nspawn/index.md
[zhs]: https://liolok.com/zhs/containerize-android-studio-with-systemd-nspawn/

## Create Btrfs Subvolume

```console
# su # switch to root user
# cd /var/lib/machines
# container_name=android-studio
# btrfs subvolume create $container_name
```

Then it will be easy to [snapshot][snapshot] and [migrate][migrate] the container.

[snapshot]: https://wiki.archlinux.org/title/Btrfs#Snapshots
[migrate]: https://wiki.archlinux.org/title/Btrfs#Send/receive

## Install Minimal Ubuntu

> Reference: [systemd-nspawn - ArchWiki](https://wiki.archlinux.org/title/Systemd-nspawn#Create_a_Debian_or_Ubuntu_environment)

Codename of current latest LTS version of Ubuntu is `focal`.

```console
# codename=focal
# repository_url='https://mirrors.bfsu.edu.cn/ubuntu/'
# debootstrap --include=systemd-container \
--components=main,universe $codename $container_name $repository_url
```

After installation, run `systemd-nspawn --machine=$container_name` to boot into brand new container.

## Network

You may need to add `--bind-ro=/etc/resolv.conf` option to make DNS work in container.

To check network status, consider commands `ping 1.1.1.1` and `ping archlinux.org`: if former works and latter doesn't, it would be DNS problem.

## 32-bit Libraries

Inside container, [enable i386 architecture][multiarch] and install the [libraries][libs].

[multiarch]: https://wiki.debian.org/Multiarch/Implementation#Using_multiarch
[libs]: https://developer.android.com/studio/install#64bit-libs

```console
# dpkg --add-architecture i386
# apt-get update
# apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386 lib32z1 libbz2-1.0:i386
```

## Android Studio

You could either `curl`/`wget` the [latest version package][download] inside container,
or download it on host and mount it to container:

[download]: https://developer.android.com/studio#downloads

```console
# host_as_archive='/data/Downloads/android-studio-ide-202.7486908-linux.tar.gz'
# systemd-nspawn --machine=$container_name --bind-ro=$host_as_archive:/tmp/as.tar.gz
```

Inside container, add a dedicated normal user named "android" and extract Android Studio to user home:

```console
# useradd --create-home android
# su - android
# tar --extract --verbose --file=/tmp/as.tar.gz
# mv android-studio studio
```

After this, Android Studio is installed under `/home/android/studio/`,
run `/home/android/studio/bin/studio.sh` to start up:

```console
# user=android
# startup_script='/home/android/studio/bin/studio.sh'
# systemd-nspawn --machine=$container_name --user=$user $startup_script
```

Oh no, this won't work because I forget to mention [Xorg stuff][xorg].
Command options become just too many, from this step we need start to
actually write up a shell script to run Android Studio in GUI.

[xorg]: https://liolok.com/run-desktop-app-with-systemd-nspawn-container/#xorg "Run Desktop App with systemd-nspawn Container"

## Fix Dependency

> TL;DR: Run `apt-get install libxext6 libxrender1 libxtst6 libxi6 libfreetype6 fontconfig` to start up.

This [log](./missing-lib.log) complains about missing library file `libXext.so.6`,
so let's try a "reverse query":

```console
# systemd-nspawn --machine=$container_name --bind-ro=/etc/resolv.conf
# apt-get install apt-file
# apt-file update
# apt-file search libXext.so.6
```

It turned out the package name should be `libxext6`. After installing it, another four same errors
came along, so all the missing libraries would be:

```console
apt-get install libxext6 libxrender1 libxtst6 libxi6 libfreetype6
```

Android Studio finally starts up... but crashes right away. The [log](./font.log) mentions "font"
a lot, so I tried installing `fontconfig` package then it finally starts up.

## Fix Cursor Theme

Mount host icon theme to container and set environment variable:

```console
# host_data=/usr/share
# data=/home/$user/.local/share
# systemd-nspawn --machine=$container_name \
--bind-ro=$host_data/icons:$data/icons --setenv=XCURSOR_PATH=$data/icons
```

Install X cursor management library in container:

```console
# systemd-nspawn --machine=$container_name --bind-ro=/etc/resolv.conf apt-get install libxcursor1
```

## Wrap Up

I wrote a [fish-shell script][script], and a [desktop entry][desktop-entry] according to [AUR][aur-ref].

[script]: https://github.com/liolok/dotfiles/blob/master/.local/bin/android-studio
[desktop-entry]: https://github.com/liolok/dotfiles/blob/master/.local/share/applications/android-studio.desktop
[aur-ref]: https://aur.archlinux.org/cgit/aur.git/tree/android-studio.desktop?h=android-studio

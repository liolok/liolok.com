# Run Desktop App with systemd-nspawn Container

## Meta Info

- Published at April 5 of 2020
- Updated at May 28 of 2021
- [Source code][source]
- [Web page][page]

[source]: https://github.com/liolok/liolok.github.io/blob/master/run-desktop-app-with-systemd-nspawn-container/index.md
[page]: https://liolok.github.io/run-desktop-app-with-systemd-nspawn-container/

## Pros

- Full filesystem isolation with manual access control;
- Independent and complete runtime, ready for migration;
- No additional dependency on host, except systemd of course.

## Cons

- Need root access to create and run container;
- OCI compatibility to pay attention?

## References

- [systemd-nspawn manual][manual]
- [systemd-nspawn - ArchWiki][archwiki]

[manual]: https://www.freedesktop.org/software/systemd/man/systemd-nspawn.html
[archwiki]: https://wiki.archlinux.org/title/Systemd-nspawn

## Choose Distribution

One could install basically almost any modern Linux distribution into a container,
the tools and instructions depend on the distributions of both host and container.

I've tried Arch Linux and Ubuntu 18.04 containers on my desktop running Arch Linux.
For more examples, refer to [examples][examples] in the manual.

[examples]: https://www.freedesktop.org/software/systemd/man/systemd-nspawn.html#Examples

### Arch Linux Container

```console
$ su # switch to root
# container_dir=/var/lib/machines # directory of containers
# mkdir $container_dir # create it if not exists yet
# cd $container_dir # change into it
# container_name=archlinux-container
# mkdir $container_name
# pacman --sync arch-install-scripts # provides `pacstrap` command
# pacstrap -c $container_name
```

About the last `pacstrap` command:

- The `-c` option tells `pacstrap` to use the package cache on the host;
- If no packages specified, `pacstrap` defaults to the `base` group;
- Additional packages could be installed like this: `pacstrap -c $container_name base base-devel sudo nano`.

### Ubuntu 18.04 Container

```console
# container_dir=/var/lib/machines # directory of my containers
# mkdir $container_dir # create it if not exists yet
# cd $container_dir # change into it
# container_name=bionic-container
# bionic_repo_url='https://mirrors.tuna.tsinghua.edu.cn/ubuntu/'
# pacman --sync debootstrap ubuntu-keyring
# debootstrap --include=systemd-container --components=main,universe bionic $container_name $bionic_repo_url
```

For `$bionic_repo_url`, one could select a suitable mirror from [Launchpad](https://launchpad.net/ubuntu/+archivemirrors).

Command `debootstrap` should finish with `I: Base system installed successfully`.
If last output is `E: Couldn't download packages:...`, just run it again to complete installation.

## Configure and Run

After acquiring a directory of minimal installation of distribution,
one can use command `systemd-nspawn -D $container_name` to start up the container,
`-D` stands for `--directory=`.

There are two easy and full-featured methods to configure a container:

- Options append to `systemd-nspawn` command;
- Configurations written in `/etc/systemd/nspawn/$container_name.nspawn` file.

At first I did try with configuration files, then I realized that,
it would be much more flexible and less hard-coded to run command in shell scripts.

### Environment Variables

`--set-env=`, `-E` in command options or `Environment=` in `[Exec]` section of configurations.

## Access Control

### Network

No command option needed, or `VirtualEthernet=no` in `[Network]` section of configurations.

### Filesystem

By default a container has no access to host filesystem.

Mount host path with `--bind=` in command options or `Bind=` in `[Files]` section of configurations;
`--bind-ro=` or `BindReadOnly=` for read-only access.

These options could be used multiple times, so that you may expose as many files/folders to container as you want.
I recommend allowing read-only access as long as it works.

I'll show several simple use cases below, with basic `--bind=`, other forms are similar.

Simplest case, expose certain file/folder on host to the **same** path in container:

```console
$ host_path=/absolute/path/on/host
# systemd-nspawn \
> --directory=$container_name \
> --bind=$host_path
```

If you want container to access it with **different** path:

```console
$ host_path=/absolute/path/on/host
$ container_path=/absolute/path/inside/container
# systemd-nspawn \
> --directory=$container_name \
> --bind=$host_path:$container_path
```

To allow one-time access, you could use an empty string for host path: `--bind=:$container_path`.
systemd-nspawn will create a temporary folder for container below the host's `/var/tmp` directory,
and remove it automatically when container is shut down.

By default, a folder will be bind-mounted recursively.
To allow access to particular path without sub-directories below it,
you could append mount option: `--bind=$host_path:$container_path:norbind`.

### Devices

For quick and full access, mount the whole `/dev/` directory on host to container.

For more fine-grained access control, consider what devices does app really need to run well,
and which files/directories under `/dev/` are the [devices allocated][linux-kernel-doc] to mount.

[linux-kernel-doc]: https://www.kernel.org/doc/html/latest/admin-guide/devices.html "Linux allocated devices (4.x+ version) - The Linux Kernel documentation"

Here is what I understood for my own usage:

| Device File    | Access to                           |
| -------------- | ----------------------------------- |
| /dev/dri/card0 | First graphics card of Intel or AMD |
| /dev/nvidia0   | First graphics card of NVIDIA       |
| /dev/shm       | Shared memory                       |
| /dev/input/js0 | First joystick                      |

Mounting device directory/file is not enough,
you also need to grant corresponding access to specific device nodes by [`DeviceAllow=`][device-allow] property.

[device-allow]: https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html#DeviceAllow=

To specific device node, use `/dev/full/path/to/device` as specifier;
for a group of device nodes, run `cat /proc/devices` to check out device group list:
- `char-` prefix for character devices;
- `block-` prefix for block devices.

For access control:
- `r` for reading;
- `w` for writing;
- `m` for creating device nodes.

For example, my AMD graphic card and joysticks:

```bash
--bind=/dev/dri/ --property=DeviceAllow='char-drm rw' # Graphic cards
--bind=/dev/input/ --property=DeviceAllow='char-input r' # Joysticks
```
### Xorg

Pass X server temporary directory and `DISPLAY` environment variable on host to container:

`--bind-ro=/tmp/.X11-unix/` and `--set-env=DISPLAY=$DISPLAY`

or in configuration file:

```properties
[Exec]
Environment=DISPLAY=:0

[Files]
BindReadOnly=/tmp/.X11-unix/
```

(Run `echo $DISPLAY` to see its value, which is usually `:0`.)

#### X Authority

As far as I experienced, if run GUI application as the same user as host,
one **may not need** to handle the authority stuff. So if window shows up after instructions above,
just skip this step.

```console
$ auth_file=/tmp/${container_name}_xauth
$ xauth nextract - "$DISPLAY" | sed -e 's/^..../ffff/' | xauth -f "$auth_file" nmerge -
# systemd-nspawn \
> --directory=$container_name \
> --bind=/tmp/.X11-unix \
> --bind="$auth_file" \
> --set-env=DISPLAY="$DISPLAY" \
> --set-env=XAUTHORITY="$auth_file"
```

## Prepare Application

Startup the container, install your application, create an unprivileged user then run it for test.
If it doesn't work as expected, go back to configure container out.

## UI Consistency

### DBus Tray

```bash
if [[ -n $DBUS_SESSION_BUS_ADDRESS ]]; then # remove prefix
    host_bus=${DBUS_SESSION_BUS_ADDRESS#unix:path=};
else # default guess
    host_bus=/run/user/$UID/bus;
fi
container_bus=/run/user/host/bus
sudo systemd-nspawn --directory=$container_name \
--bind-ro=$host_bus:$container_bus \
--set-env=DBUS_SESSION_BUS_ADDRESS=unix:path=$container_bus
```

### PulseAudio

```bash
if [[ -n $PULSE_SERVER ]]; then # remove prefix
  host_pulse=${PULSE_SERVER#unix:};
else # default guess
  host_pulse=/run/user/$UID/pulse;
fi
container_pulse=/run/user/host/pulse/
systemd-nspawn --direcoty=$container_name \
--bind-ro=$host_pulse:$container_pulse \
--setenv=PULSE_SERVER=unix:$container_pulse/native
```

Note that: Apps that explicitly depend on ALSA could break PulseAudio.
For Arch Linux container `pacman --sync pulseaudio-alsa --assume-installed pulseaudio` solves the problem.

### Styles

If app starts up but looks different than how it does on host. Mounting themes,
icons and fonts to container would help:

```bash
--bind-ro=/usr/share/themes/:/home/container_username/.local/share/themes/
--bind-ro=/usr/share/icons/:/home/container_username/.local/share/icons/
--bind-ro=/usr/share/fonts/:/home/container_username/.local/share/fonts/
```

Note that the user directories like `~/.cache`, `~/.config` and `~/.local/share` in container need to be existing before mounting,
otherwise applications would probably fail to write into them.

For font configuration:

```bash
--bind-ro=/home/host_username/.config/fontconfig/:/home/container_username/.config/fontconfig/
```

For cursor icon search path:

```bash
--setenv=XCURSOR_PATH=/home/container_username/.local/share/icons
```

These two commands would be useful to manually refresh cache in container:

```bash
fc-cache --force
gtk-update-icon-cache --force
```

## Ready to Go

After test, container should be ready for app. Write a shell script and wrap it into a desktop entry file,
so it looks like a native app then.

### Shell Script

`~/.local/bin/app.bash`:

```bash
#!/bin/bash

# Path to container
container_name=your-container
container_path=/var/lib/machines # container parent directory
container_path+=/$container_name # container root directory

# Binary of target application
app_binary=/path/to/your/application

# Run container directly into target application
pkexec systemd-nspawn \
--directory=$container_path \
--user=$USER --chdir=$HOME \
--bind-ro=/tmp/.X11-unix/ \
--setenv=DISPLAY=$DISPLAY \
--as-pid2 $app_binary
```

Tool `pkexec` is used for root authorization under GUI,
feel free to replace with `sudo` if doesn't require password.

See also: my fish script to run steam: [~/.local/bin/steam][steam].

[steam]: https://github.com/liolok/dotfiles/blob/master/.local/bin/steam

### Desktop Entry

Reference: [Desktop entries (.desktop)](https://www.freedesktop.org/wiki/Specifications/desktop-entry-spec/)

`~/.local/share/applications/app.desktop`:

```properties
[Desktop Entry]
Type=Application
Name=Your App
Comment=Description of your app
Exec=app.bash
Icon=icon-file-name
```

For `Exec` key, script file name `app.bash` could be used only after adding its parent folder to your `PATH` environment variable,
otherwise input its full path `/home/host_username/.local/bin/app.bash`. Former method is also convenient for running from terminal.

For [`Icon`][desktop_icon] key, put custom icon file into `$XDG_DATA_DIRS/icons/hicolor/[size]/apps/`.

[desktop_icon]: https://specifications.freedesktop.org/icon-theme-spec/icon-theme-spec-latest.html

See also: my desktop entry to run steam: [~/.local/share/applications/steam.desktop][steam.desktop].

[steam.desktop]: https://github.com/liolok/dotfiles/blob/master/.local/share/applications/steam.desktop

# Set Environment Variables with pam_env

- Published: 2020-08-10
- [Markdown][raw]

[raw]: https://raw.githubusercontent.com/liolok/liolok.com/master/set-environment-variables-with-pam-env/index.md

**Warning**: Upstream totally deprecated user level configuration, this article is reserved only
for reference.

I hate pam_env, its official documentation isn't consistent with actual behavior, and oldest code is
even older than me.

For now I set my environment variables using fish (my login shell), and use `startx startplasma-x11`
to startup KDE. Refer to my [fish configurations][fish_conf].

[fish_conf]: https://github.com/liolok/dotfiles/tree/master/.config/fish/conf.d

## Pros

It's a PAM module and sets environment variables as soon as user logs in.

Bash, zsh or fish; Wayland or Xorg; no need to bother understanding their
environment scopes, not as global as pam_env anyway.

## Cons

- May be too global if want set environment variable in certain scope like only under shell or GUI;
- Configuration files are pure text, not any kind of shell script;
- Require root permission since version 1.4.0 (**Or** count on default configuration of distro).

## Configuration Files

`pam_env` reads these three files below in turn:

- `/etc/security/pam_env.conf`: This file need to be correctly parsed (empty is fine),
or the other two files won't be parsed at all.
- `/etc/environment`: Simple `[export ]KEY=VAL` syntax, no spaces escaping, `#` for comments.
- `~/.pam_environment`: User-level `pam_env.conf`, disabled by default since v1.4.0.

### User-Level Configuration

Since PAM **v1.4.0**, pam_env module [doesn't read user-level configuration file by default][commit].
To enable user-level config, one has to be administrator to append argument in system-level config of PAM.

[commit]: https://github.com/linux-pam/linux-pam/commit/f83fb5
"pam_env: Change the default to not read the user .pam_environment file · linux-pam/linux-pam@f83fb5f"

Run `grep pam_env.so /etc/pam.{conf,d/*}` to see all the references of pam_env module,
file results depend on distros and personal use case.

On Gentoo and Arch Linux it's `/etc/pam.d/system-login` to be altered,
go to the `session required pam_env.so` line and append argument `user_readenv=1`.

(Note for Arch Linux users: `pambase` package of [has already done the work][pambase_commit] above.)

[pambase_commit]:
https://github.com/archlinux/svntogit-packages/commit/2d5af94ae55a5c98837ce9631f331ad2aad32bb3#diff-02a81ec7974f6d6a00a87a0d9ce507b6R19
"Remove options not supported by faillock, Drop sha512 option to pam_u… · archlinux/svntogit-packages@2d5af94 · GitHub"

Additionally, to custom file path instead of default `~/.pam_environment`,
append argument `user_envfile=path`, in which the `path` is relative to **every** user's home.

The altered line may look like:
```
session    required    pam_env.so user_readenv=1 user_envfile=.config/pam_env.conf
```

## Configuration Syntax

*VARIABLE_NAME* [**DEFAULT=**[*value*]] [**OVERRIDE=**[*value*]]

For quick and simplest usage: `VARIABLE_NAME DEFAULT=value` *usually* works like bash command `export VARIABLE_NAME=value`.

### Priority Mechanism

1. Override value if provided and expanded to non-empty string;
2. Undefined (deleted) if literally `DEFAULT=` provided;
3. Default value including empty string expanded or provided with `""`.

### Expanding

For defined values:

- Use `${VARIABLE_NAME}` to expand variables;
- Use `@{PAM_ITEM_NAME}` to expand PAM_ITEMs including:
    - PAM_USER
    - PAM_USER_PROMPT
    - PAM_TTY
    - PAM_RUSER
    - PAM_RHOST

Special variables `@{HOME}` and `@{SHELL}` are expanded to values for user according
to its record in user database, which is `/etc/passwd` file in most cases.

### Escaping

- Use `\$` and `\@` for their literal values;
- Use `"value containing spaces"` to escape spaces, escaped `"` not supported;
- Use backslash at end of line to escape newlines `\n`;
- Use `#` to start comments anywhere, escaped `#` not supported.

### Examples

(These examples are from Dave Kinchlea, original author of pam_env module.)

Set the `REMOTEHOST` variable for any hosts that are remote,
default to "localhost" rather than not being set at all:
```
REMOTEHOST    DEFAULT=localhost    OVERRIDE=@{PAM_RHOST}
```

Set the DISPLAY variable if it seems reasonable:
```
DISPLAY    DEFAULT=${REMOTEHOST}:0.0    OVERRIDE=${DISPLAY}
```

Now some simple variables:
```
PAGER       DEFAULT=less
MANPAGER    DEFAULT=less
LESS        DEFAULT="M q e h15 z23 b80"
NNTPSERVER  DEFAULT=localhost
PATH        DEFAULT=${HOME}/bin:/usr/local/bin:/bin\
:/usr/bin:/usr/local/bin/X11:/usr/bin/X11
```

Silly examples of escaped variables, just to show how they work:
```
DOLLAR          DEFAULT=\$
DOLLARDOLLAR    DEFAULT=    OVERRIDE=\$${DOLLAR}
DOLLARPLUS      DEFAULT=\${REMOTEHOST}${REMOTEHOST}
ATSIGN          DEFAULT=""  OVERRIDE=\@
```

## References

### Source Code

*[linux-pam/pam_env.c at v1.4.0 · linux-pam/linux-pam · GitHub](https://github.com/linux-pam/linux-pam/blob/v1.4.0/modules/pam_env/pam_env.c)*

### Documentation

Linux-PAM: *[6.6. pam_env - set/unset environment variables](http://www.linux-pam.org/Linux-PAM-html/sag-pam_env.html)*

<!-- Confusing example added at [this commit][pam_env.conf], it would not work according to source code. -->

[pam_env.conf]: https://github.com/linux-pam/linux-pam/commit/73bdfac8c091492f466342feb8f2f5daa2f4c39b#diff-95172cf33f38cd831c4172f676ec6d18R97
"pam_env: expand @{HOME} and @{SHELL} and enhance documentation · linux-pam/linux-pam@73bdfac · GitHub"

Arch Linux: [Environment variables - ArchWiki](https://wiki.archlinux.org/index.php/Environment_variables#Using_pam_env)

Ubuntu: [EnvironmentVariables - Community Help Wiki](https://help.ubuntu.com/community/EnvironmentVariables#A.2BAH4-.2F.pam_environment)

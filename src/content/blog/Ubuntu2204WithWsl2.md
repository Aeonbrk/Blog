---
title: Ubuntu WSL2
author: Haibin
pubDatetime: 2023-12-24T13:50:00+8:00
featured: false
draft: false
tags:
  - Linux
  - WSL2
ogImage: ""
description: A configuration for Ubuntu WSL2.
---

## Table of contents

## Problems

### Access is denied

```shell
D:\ ❯ wsl --shutdown
D:\ ❯ wsl -l -v
  NAME                   STATE           VERSION
* Ubuntu-22.04           Stopped         2
  docker-desktop-data    Stopped         2
  docker-desktop         Stopped         2
  Ubuntu-20.04           Stopped         2
D:\ ❯ wsl --export Ubuntu-22.04 H:\wsl
Access is denied.
Error code: Wsl/E_ACCESSDENIED
D:\ ❯ wsl --export Ubuntu-22.04 H:\wsl\Ubuntu2204.tar
Export in progress, this may take a few minutes...
```

- you should explicitly give the backup a name

## apt source

1. configure default user
   `ubuntu2204 config --default-user  haibin`

2. change default apt source
   `sudo vim /etc/apt/sources.list`

**ubuntu 20.04 (focal) 配置如下**

```shell
deb https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

## deb https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
## deb-src https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe
```

**ubuntu 22. 04 (jammy) 配置如下**

```shell
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
```

**default**

```shell
deb http://archive.ubuntu.com/ubuntu/ jammy main restricted
deb http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted
deb http://archive.ubuntu.com/ubuntu/ jammy universe
deb http://archive.ubuntu.com/ubuntu/ jammy-updates universe
deb http://archive.ubuntu.com/ubuntu/ jammy multiverse
deb http://archive.ubuntu.com/ubuntu/ jammy-updates multiverse
deb http://archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted
deb http://security.ubuntu.com/ubuntu/ jammy-security universe
deb http://security.ubuntu.com/ubuntu/ jammy-security multiverse
```

`sudo apt update && sudo apt upgrade`

## proxy set

```shell
#!/bin/sh
hostip=$(cat /etc/resolv.conf | grep nameserver | awk '{ print $2 }')
wslip=$(hostname -I | awk '{print $1}')
port=7890

PROXY_HTTP="http://${hostip}:${port}"

set_proxy(){
  export http_proxy="${PROXY_HTTP}"
  export HTTP_PROXY="${PROXY_HTTP}"

  export https_proxy="${PROXY_HTTP}"
  export HTTPS_proxy="${PROXY_HTTP}"

  export ALL_PROXY="${PROXY_SOCKS5}"
  export all_proxy=${PROXY_SOCKS5}

  git config --global http.https://github.com.proxy ${PROXY_HTTP}
  git config --global https.https://github.com.proxy ${PROXY_HTTP}

  echo "Host IP:" ${hostip}
  echo "WSL IP:" ${wslip}
  echo "http_proxy:" ${PROXY_HTTP}
  echo "Try to connect to Google..."
  resp=$(curl -I -s --connect-timeout 5 -m 5 -w "%{http_code}" -o /dev/null www.google.com)
  if [ ${resp} = 200 ]; then
    echo "Proxy setup succeeded!"
  else
    echo "Proxy setup failed!"
  fi

}

unset_proxy(){
  unset http_proxy
  unset HTTP_PROXY
  unset https_proxy
  unset HTTPS_PROXY
  unset ALL_PROXY
  unset all_proxy
  git config --global --unset http.https://github.com.proxy
  git config --global --unset https.https://github.com.proxy

  echo "Proxy has been closed."
}


if [ "$1" = "set" ]
then
  set_proxy

elif [ "$1" = "unset" ]
then
  unset_proxy

else
  echo "Unsupported arguments."
fi
```

`in ~/. bashrc or ~/. zshrc`

```shell
# alias
alias r='ranger'
alias v='lvim'
alias vi='lvim'
alias vim='lvim'
alias f='fzf'
alias c='bat'
alias proxy="source ~/proxy.sh"
alias clean='sudo bleachbit'
alias pdf='evince'
alias u='sudo apt update && sudo apt upgrade'

# proxy
proxy set
```

## git

- [Generating a new SSH key and adding it to the ssh-agent - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=windows)
- [Adding a new SSH key to your GitHub account - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

## nvim

```shell
sudo apt-get install ninja-build gettext cmake unzip curl build-essential
git clone https://github.com/neovim/neovim
cd neovim && git checkout stable
```

If a package is not provided for your platform, you can build Neovim from source. See [Building-Neovim](https://github.com/neovim/neovim/wiki/Building-Neovim) for details. If you have the [prerequisites](https://github.com/neovim/neovim/wiki/Building-Neovim#build-prerequisites) then building is easy:

```cpp
make CMAKE_BUILD_TYPE=Release -j8
sudo make install -j8
```

> For Unix-like systems this installs Neovim to `/usr/local`, while for Windows to `C:\Program Files`. Note, however, that this can complicate uninstallation. The following example avoids this by isolating an installation under `$HOME/neovim`:
>
> ```cpp
> rm -r build/  # clear the CMake cache
> make CMAKE_EXTRA_FLAGS="-DCMAKE_INSTALL_PREFIX=$HOME/neovim"
> make install
> export PATH="$HOME/neovim/bin:$PATH"
> ```

### Lunarvim

[Installation | LunarVim](https://www.lunarvim.org/docs/installation)

- Optinal: node and Rust

```shell
sudo apt update && sudo apt install git make python3-pip

# install
LV_BRANCH='release-1.3/neovim-0.9' bash <(curl -s https://raw.githubusercontent.com/LunarVim/LunarVim/release-1.3/neovim-0.9/utils/installer/install.sh)

sudo apt install python3-venv xclip
pip3 install neovim
npm install -g neovim

# uninstall
bash ~/.local/share/lunarvim/lvim/utils/installer/uninstall.sh
```

```shell
cd ~\.config\lvim && mkdir lua && mkdir ftplugin
cd lua && mkdir user
```

```shell
cd ~\.config\lvim\ftplugin
```

## oh my zsh

> 1. [Install the recommended font](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k). *Optional but highly recommended.*
> 2. [Install Powerlevel10k](https://github.com/romkatv/powerlevel10k#installation) itself.
> 3. Restart Zsh with `exec zsh`.
> 4. Type `p10k configure` if the configuration wizard doesn't start automatically.

[Installing ZSH · ohmyzsh/ohmyzsh Wiki (github.com)](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH)

```shell
sudo apt install zsh
zsh --version
chsh -s $(which zsh)
```

[Home · ohmyzsh/ohmyzsh Wiki (github.com)](https://github.com/ohmyzsh/ohmyzsh/wiki)

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### powerlevel 10 k theme

[romkatv/powerlevel10k: A Zsh theme (github.com)](https://github.com/romkatv/powerlevel10k)

```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
ZSH_THEME="powerlevel10k/powerlevel10k"   # in ~/.zshrc
```

### plugins

```shell
# zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# in zshrc
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

## fzf

```shell
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

### Key bindings for command-line

The install script will setup the following key bindings for bash, zsh, and fish.

- `CTRL-T` - Paste the selected files and directories onto the command-line

  - Set `FZF_CTRL_T_COMMAND` to override the default command
  - Set `FZF_CTRL_T_OPTS` to pass additional options to fzf

  ```shell
  # Preview file content using bat (https://github.com/sharkdp/bat)
  export FZF_CTRL_T_OPTS="
    --preview 'bat -n --color=always {}'
    --bind 'ctrl-/:change-preview-window(down|hidden|)'"
  ```

- `CTRL-R` - Paste the selected command from history onto the command-line

  - If you want to see the commands in chronological order, press `CTRL-R` again which toggles sorting by relevance
  - Set `FZF_CTRL_R_OPTS` to pass additional options to fzf

  ```shell
  # CTRL-/ to toggle small preview window to see the full command
  # CTRL-Y to copy the command into clipboard using pbcopy
  export FZF_CTRL_R_OPTS="
    --preview 'echo {}' --preview-window up:3:hidden:wrap
    --bind 'ctrl-/:toggle-preview'
    --bind 'ctrl-y:execute-silent(echo -n {2..} | pbcopy)+abort'
    --color header:italic
    --header 'Press CTRL-Y to copy command into clipboard'"
  ```

- `ALT-C` - cd into the selected directory

  - Set `FZF_ALT_C_COMMAND` to override the default command
  - Set `FZF_ALT_C_OPTS` to pass additional options to fzf

  ```shell
  # Print tree structure in the preview window
  export FZF_ALT_C_OPTS="--preview 'tree -C {}'"
  ```

If you're on a tmux session, you can start fzf in a tmux split-pane or in a tmux popup window by setting `FZF_TMUX_OPTS` (e.g. `export FZF_TMUX_OPTS='-p80%,60%'`). See `fzf-tmux --help` for available options.

More tips can be found on [the wiki page](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings).

### fzf-tab

[Aloxaf/fzf-tab: Replace zsh's default completion selection menu with fzf! (github.com)](https://github.com/Aloxaf/fzf-tab)

Clone this repository to your custom directory and then add `fzf-tab` to your plugin list.

```shell
git clone https://github.com/Aloxaf/fzf-tab ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/fzf-tab
```

## node/npm/nvm

To **install** or **update** nvm, you should run the [install script](https://github.com/nvm-sh/nvm/blob/v0.39.3/install.sh). To do that, you may either download and run the script manually, or use the following cURL or Wget command:

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

# or

wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

Running either of the above commands downloads a script and runs it. The script clones the nvm repository to `~/.nvm`, and attempts to add the source lines from the snippet below to the correct profile file (`~/.bash_profile`, `~/.zshrc`, `~/.profile`, or `~/.bashrc`).

```shell
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

```shell
nvm list-remote
nvm install --lts

# after type nvm install --lts
Installing latest LTS version.
Downloading and installing node v18.16.0...
Local cache found: ${NVM_DIR}/.cache/bin/node-v18.16.0-linux-x64/node-v18.16.0-linux-x64.tar.xz
Computing checksum with sha256sum
Checksums do not match: 'fb19b07e7e9df431af7b4762c3800d21f7eb9ce489b459155cad4648068ea422' found, '44d93d9b4627fe5ae343012d855491d62c7381b236c347f7666a7ad070f26548' expected.
Checksum check failed!
Removing the broken local cache...
Downloading https://nodejs.org/dist/v18.16.0/node-v18.16.0-linux-x64.tar.xz...
####################################################################################### 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v18.16.0 (npm v9.5.1)
Creating default alias: default -> lts/* (-> v18.16.0)
❯ node -v
v18.16.0
❯ npm -v
9.5.1
```

## Rust/cargo

[Installation - The Cargo Book (rust-lang.org)](https://doc.rust-lang.org/cargo/getting-started/installation.html)

The easiest way to get Cargo is to install the current stable release of [Rust](https://www.rust-lang.org/) by using [rustup](https://rustup.rs/). Installing Rust using `rustup` will also install `cargo`.

On Linux and macOS systems, this is done as follows:

`curl https://sh.rustup.rs -sSf | sh`

It will download a script, and start the installation. If everything goes well, you'll see this appear:

`Rust is installed now. Great!`

## Tmux

[A Quick and Easy Guide to tmux (hamvocke.com)](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)
[Linux Command Line Adventure: Terminal Multiplexers](http://linuxcommand.org/lc3_adv_termmux.php)

## dotfiles

[GitHub does dotfiles - dotfiles.github.io](https://dotfiles.github.io/)
[mathiasbynens/dotfiles: .files, including ~/.macos — sensible hacker defaults for macOS (github.com)](https://github.com/mathiasbynens/dotfiles)

## The missing semester

- [Vim · the missing semester of your cs education (missing-semester-cn.github.io)](https://missing-semester-cn.github.io/2020/editors/)
- [Command-Line · the missing semester of your cs education (missing-semester-cn.github.io)](https://missing-semester-cn.github.io/2020/command-line/)
- [Debugging-Profiling · the missing semester of your cs education (missing-semester-cn.github.io)](https://missing-semester-cn.github.io/2020/debugging-profiling/)
- [Security · the missing semester of your cs education (missing-semester-cn.github.io)](https://missing-semester-cn.github.io/2020/security/)
- [Potpourri · the missing semester of your cs education (missing-semester-cn.github.io)](https://missing-semester-cn.github.io/2020/potpourri/)

## teminalizer

> [faressoft/terminalizer: 🦄 Record your terminal and generate animated gif images or share a web player (github.com)](https://github.com/faressoft/terminalizer)
>
> Record your terminal and generate animated gif images or share a web player link [terminalizer.com](https://terminalizer.com/)

![Alt text](https://github.com/faressoft/terminalizer/raw/master/img/demo.gif?raw=true "optional title")

## Potpourri

[[Modern Unix]]
[Install Google Chrome on Ubuntu 22.04/20.04 From Command Line (linuxbabe.com)](https://www.linuxbabe.com/ubuntu/install-google-chrome-ubuntu-20-04)

### thefuck

[nvbn/thefuck: Magnificent app which corrects your previous console command. (github.com)](https://github.com/nvbn/thefuck)

> *he Fuck* is a magnificent app, inspired by a [@liamosaur](https://twitter.com/liamosaur/) [tweet](https://twitter.com/liamosaur/status/506975850596536320), that corrects errors in previous console commands.
>
> Is *The Fuck* too slow? [Try the experimental instant mode!](https://github.com/nvbn/thefuck#experimental-instant-mode)

```shell
sudo apt update
sudo apt install python3-dev python3-pip python3-setuptools
pip3 install thefuck --user
```

### bat

[bat/README-zh.md at master · sharkdp/bat · GitHub](https://github.com/sharkdp/bat/blob/master/doc/README-zh.md)

#### Ubuntu (use apt)

`bat` request version： [Ubuntu after 20.04 ("Focal")](https://packages.ubuntu.com/search?keywords=bat&exact=1) 和 [Debian after August 2021 (Debian 11 - "Bullseye")](https://packages.debian.org/bullseye/bat).
if your distribution is satisfy the requests：

```shell
sudo apt install bat
```

Important: if you are in this way to install `bat`, please pay attention to whether or not you have installed executables for `batcat` (by [other bundles executable file name conflict caused](https://github.com/sharkdp/bat/issues/982)) . You can create a 'bat -> batcat' symlink or alias to avoid problems due to different executables and maintain consistency with other distributions.

```shell
mkdir -p ~/.local/bin
ln -s /usr/bin/batcat ~/.local/bin/bat
```

#### Ubuntu (使用 .deb 包)

If you cannot install using the previous method, or need the latest version of bat, you can download the latest.deb package from [release page](https://github.com/sharkdp/bat/releases) and install it as follows:

```shell
wget https://github.com/sharkdp/bat/releases/download/v0.23.0/bat-musl_0.23.0_amd64.deb
sudo dpkg -i bat-musl_0.23.0_amd64.deb  # adapt version number and architecture
```

### fd

> `fd` is a program to find entries in your filesystem. It is a simple, fast and user-friendly alternative to [`find`](https://www.gnu.org/software/findutils/). While it does not aim to support all of `find` 's powerful functionality, it provides sensible (opinionated) defaults for a majority of use cases.

#### On Ubuntu

If you run Ubuntu 19.04 (Disco Dingo) or newer, you can install the [officially maintained package](https://packages.ubuntu.com/fd-find)

```shell
sudo apt install fd-find
```

Note that the binary is called `fdfind` as the binary name `fd` is already used by another package. It is recommended that after installation, you add a link to `fd` by executing command `ln -s $(which fdfind) ~/.local/bin/fd`, in order to use `fd` in the same way as in this documentation. Make sure that `$HOME/.local/bin` is in your `$PATH`.

If you use an older version of Ubuntu, you can download the latest `.deb` package from the [release page](https://github.com/sharkdp/fd/releases) and install it via:

```shell
wget https://github.com/sharkdp/fd/releases/download/v8.7.0/fd-musl_8.7.0_amd64.deb
sudo dpkg -i fd-musl_8.7.0_amd64.deb  # adapt version number and architecture
```

[junegunn/fzf: A command-line fuzzy finder (github.com)](https://github.com/junegunn/fzf#tips)

### ccache

> Ccache is a compiler cache. It [speeds up recompilation](https://ccache.dev/performance.html) by caching previous compilations and detecting when the same compilation is being done again.
> [ccache/ccache: ccache – a fast compiler cache (github.com)](https://github.com/ccache/ccache)

[ccache package versions - Repology](https://repology.org/project/ccache/versions)

```shell
sudo apt install ccache
which gcc
```

By default, it outputs' /usr/bin/gcc ', meaning that when you execute the 'gcc' command, you are actually executing '/usr/bin/gcc'. As an exercise in RTFM, you will then need to read the contents of man ccache and set an environment variable correctly in the.zshrc file according to the manual instructions. If your setup is successful, rerun 'which gcc' and you will see the output changed to '/usr/lib/ccache/gcc'.

```shell
 There are two ways to use ccache. You can either prefix your compilation
 commands with ccache or you can let ccache masquerade as the compiler by
 creating a symbolic link (named as the compiler) to ccache. The first method is
 most convenient if you just want to try out ccache or wish to use it for some
 specific projects. The second method is most useful for when you wish to use
 ccache for all your compilations.

 To use the first method, just make sure that ccache is in your PATH.

 To use the second method on a Debian system, it’s easiest to just prepend
 /usr/lib/ccache to your PATH. /usr/lib/ccache contains symlinks for all
 compilers currently installed as Debian packages.

   Alternatively, you can create any symlinks you like yourself like this:

     ln -s /usr/bin/ccache /usr/local/bin/gcc
     ln -s /usr/bin/ccache /usr/local/bin/g++
     ln -s /usr/bin/ccache /usr/local/bin/cc
     ln -s /usr/bin/ccache /usr/local/bin/c++

 And so forth. This will work as long as the directory with symlinks comes before
 the path to the compiler (which is usually in /usr/bin). After installing you
 may wish to run “which gcc” to make sure that the correct link is being used.
```

```shell
vi ~/.zshrc
export PATH="/usr/lib/ccache:$PATH"
source ~/.zshrc
which gcc
```

### btop

> [aristocratos/btop: A monitor of resources (github.com)](https://github.com/aristocratos/btop)
> Resource monitor that shows usage and stats for processor, memory, disks, network and processes.

Needs GCC 10 or higher, (GCC 11 or above strongly recommended for better CPU efficiency in the compiled binary). The makefile also needs GNU coreutils and `sed` (should already be installed on any modern distribution).
For a `cmake` based build alternative see the [fork](https://github.com/jan-guenter/btop/tree/main) by @jan-guenter

1. **Install dependencies (example for Ubuntu 21.04 Hirsute)**
   Use gcc-10 g++-10 if gcc-11 isn't available

   ```shell
   sudo apt install coreutils sed git build-essential gcc-11 g++-11
   ```

2. **Clone repository**

   ```shell
   git clone https://github.com/aristocratos/btop.git
   cd btop
   ```

3. **Compile**

   ```shell
   Append `VERBOSE=true` to display full compiler/linker commands.
   Append `STATIC=true` for static compilation.
   Notice! If using LDAP Authentication, usernames will show as UID number for LDAP users if compiling statically with glibc.
   Append `QUIET=true` for less verbose output.
   Append `STRIP=true` to force stripping of debug symbols (adds `-s` linker flag).
   Append `ARCH=<architecture>` to manually set the target architecture. If omitted the makefile uses the machine triple (output of `-dumpmachine` compiler parameter) to detect the target system.
   Use `ADDFLAGS` variable for appending flags to both compiler and linker.
   For example: `ADDFLAGS=-march=native` might give a performance boost if compiling only for your own system.
   If `g++` is linked to an older version of gcc on your system specify the correct version by appending `CXX=g++-10` or `CXX=g++-11`.

   make VERBOSE=true -j12
   ```

4. **Install**
   Append `PREFIX=/target/dir` to set target, default: `/usr/local`
   Notice! Only use "sudo" when installing to a NON user owned directory.

   ```shell
   sudo make install
   ```

### zoxide

> [ajeetdsouza/zoxide: A smarter cd command. Supports all major shells. (github.com)](https://github.com/ajeetdsouza/zoxide)
> zoxide is a **smarter cd command**, inspired by z and autojump. It remembers which directories you use most frequently, so you can "jump" to them in just a few keystrokes. zoxide works on all major shells.

```shell
curl -sS https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | bash

# or

sudo apt install zoxide
```

```shell
zoxide init --cmd z zsh
```

Add this to the **end** of your config file (usually `~/.zshrc`):

```shell
eval "$(zoxide init zsh)"
```

For completions to work, the above line must be added *after* `compinit` is called. You may have to rebuild your completions cache by running `rm ~/.zcompdump*; compinit`.

### typos

> [crate-ci/typos: Source code spell checker (github.com)](https://github.com/crate-ci/typos)
> Finds and corrects spelling mistakes among source code:
>
> - Fast enough to run on monorepos
> - Low false positives so you can run on PRs

### tldr

> [tldr-pages/tldr: 📚 Collaborative cheatsheets for console commands (github.com)](https://github.com/tldr-pages/tldr)
> The **tldr-pages** project is a collection of community-maintained help pages for command-line tools, that aims to be a simpler, more approachable complement to traditional [man pages](https://en.wikipedia.org/wiki/Man_page).

A popular and convenient way to access these pages on your computer is to install the [Node.js client](https://github.com/tldr-pages/tldr-node-client), which is supported by the tldr-pages project maintainers:

```shell
npm install -g tldr
```

### gping

Ping, but with a graph.

Comes with the following super-powers:

- Graph the ping time for multiple hosts
- Graph the *execution time* for commands via the `--cmd` flag
- Custom colours
- Windows, Mac and Linux support

```shell
echo "deb http://packages.azlux.fr/debian/ buster main" | sudo tee /etc/apt/sources.list.d/azlux.list
wget -qO - https://azlux.fr/repo.gpg.key | sudo apt-key add -
sudo apt update
sudo apt install gping

gping --help
```

### ranger

```shell
❯ ranger --copy-config=all
creating: /home/haibin/.config/ranger/rifle.conf
creating: /home/haibin/.config/ranger/commands.py
creating: /home/haibin/.config/ranger/commands_full.py
creating: /home/haibin/.config/ranger/rc.conf
creating: /home/haibin/.config/ranger/scope.sh

> Please note that configuration files may change as ranger evolves.
  It's completely up to you to keep them up to date.

> To stop ranger from loading both the default and your custom rc.conf,
  please set the environment variable RANGER_LOAD_DEFAULT_RC to FALSE.
```

### cht.sh

[chubin/cheat.sh: the only cheat sheet you need (github.com)](https://github.com/chubin/cheat.sh)

Unified access to the best community driven cheat sheets repositories of the world.
Let's imagine for a moment that there is such a thing as an ideal cheat sheet. What should it look like? What features should it have?

- **Concise** — It should only contain the things you need, and nothing else.
- **Fast** — It should be possible to use it instantly.
- **Comprehensive** — It should contain answers for every possible question.
- **Universal** — It should be available everywhere, anytime, without any preparations.
- **Unobtrusive** — It should not distract you from your main task.
- **Tutoring** — It should help you to learn the subject.
- **Inconspicuous** — It should be possible to use it completely unnoticed.

Such a thing exists! It's easy to [install](https://github.com/chubin/cheat.sh#installation) and there's even [auto-complete](https://github.com/chubin/cheat.sh#tab-completion).

```shell
# installation
PATH_DIR="$HOME/bin"  # or another directory on your $PATH
mkdir -p "$PATH_DIR"
curl https://cht.sh/:cht.sh > "$PATH_DIR/cht.sh"
chmod +x "$PATH_DIR/cht.sh"
```

### gdu

Pretty fast disk usage analyzer written in Go.

Gdu is intended primarily for SSD disks where it can fully utilize parallel processing. However HDDs work as well, but the performance gain is not so huge.

```shell
sudo add-apt-repository ppa:daniel-milde/gdu
sudo apt-get update
sudo apt-get install gdu
```

### fkill-cli

[sindresorhus/fkill-cli: Fabulously kill processes. Cross-platform. (github.com)](https://github.com/sindresorhus/fkill-cli)

> Fabulously kill processes. Cross-platform.

Works on macOS, Linux, and Windows.

```shell
npm install --global fkill-cli
```

**Usage**

```shell
$ fkill --help

Usage
	$ fkill [<pid|name|:port> …]
Options
	--force, -f                  Force kill
	--verbose, -v                Show process arguments
	--silent, -s                 Silently kill and always exit with code 0
	--force-timeout <N>, -t <N>  Force kill processes which didn't exit after N seconds
Examples
	$ fkill 1337
	$ fkill safari
	$ fkill :8080
	$ fkill 1337 safari :8080
	$ fkill
To kill a port, prefix it with a colon. For example: :8080.
Run without arguments to use the interactive interface.
In interactive mode, 🚦n% indicates high CPU usage and 🐏n% indicates high memory sage.
Supports fuzzy search in the interactive mode.
The process name is case insensitive.
```

### gdb-dashboard

[cyrus-and/gdb-dashboard: Modular visual interface for GDB in Python (github.com)](https://github.com/cyrus-and/gdb-dashboard)

> GDB dashboard is a standalone `.gdbinit` file written using the [Python API](https://sourceware.org/gdb/onlinedocs/gdb/Python-API.html) that enables a modular interface showing relevant information about the program being debugged. Its main goal is to reduce the number of GDB commands needed to inspect the status of current program thus allowing the developer to primarily focus on the control flow.

**in your home directory**

```shell
wget -P ~ https://git.io/.gdbinit
```

### gcalcli

> gcalcli is a Python application that allows you to access your Google Calendar(s) from a command line. It's easy to get your agenda, search for events, add new events, delete events, edit events, see recently updated events, and even import those annoying ICS/vCal invites from Microsoft Exchange and/or other sources. Additionally, gcalcli can be used as a reminder service and execute any application you want when an event is coming up.

```shell
sudo apt install gcalcli
```

### mapscii

[rastapasta/mapscii: 🗺 MapSCII is a Braille & ASCII world map renderer for your console - enter => telnet mapscii.me <= on Mac (brew install telnet) and Linux, connect with PuTTY on Windows (github.com)](https://github.com/rastapasta/mapscii)

> A node.js based [Vector Tile](http://wiki.openstreetmap.org/wiki/Vector_tiles) to [Braille](http://www.fileformat.info/info/unicode/block/braille_patterns/utf8test.htm) and [ASCII](https://de.wikipedia.org/wiki/American_Standard_Code_for_Information_Interchange) renderer for [xterm](https://en.wikipedia.org/wiki/Xterm) -compatible terminals.

If you haven't already got Node.js >= version 10, then [go get it](http://nodejs.org/).

```shell
npm install -g mapscii
```

This is pretty simple too.

```shell
mapscii
```

---

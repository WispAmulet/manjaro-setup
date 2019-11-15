# 记录安装使用 Manjaro with i3wm 的过程以及遇到的问题

**_Upgrade-2019-11-15:_**

> `当使用 mod+0 u 切换用户时`，系统卡死，重启无法登录。更换 DM 为 SDDM 后可以登录，但切回系统默认的 LDM 时仍然无法登录

> 使用两天后在 DM 界面输入正确的密码无法登录，提示为 `Login Failed`。切换到 tty2 可以以 root 用户登录，磁盘使用情况正常，可以正常访问文件。但仍然无法以普通用户登录，提示为 `Incorrect login`。回忆之前的操作如下：1. `安装 Simple Terminal 并设置为默认` 2. `切换默认的 shell 为 zsh` 3. `还有一件事我忘了`。解决办法：重装 😭😭😭

## 安装之前

1. 在官网下载 `iso` 文件，Debian 环境下使用 `dd` 命令写入到 U 盘，重启后进入 UEFI ，启动项中找不到该 U 盘

   解决：在 Windows 10 中使用 Rufus 刻录到 U 盘，刻录模式选择 `dd`

2. 进入 Manjaro Live 环境后，直接安装黑屏或自动关机，命令行出现 `A start job is running for livemedia mhwd scripe(xxxx)` 之类的提示

   Q: 可能是笔记本双显卡的原因

   A: 重新启动进入 Live 环境，移动到 boot 项时按 e 编辑，在命令行中把 `driver=free` 修改为 `driver=intel`，并在最后添加 `xdriver=mesa acpi_osi=! acpi_osi="Windows 2009"`。

   ~~TODO: 网上有人遇到按此方法可以正常安装，但重启之后无法进入系统的情况，需要修改 grub 文件。本人测试后没有遇到，猜测可能是因为双系统的原因(?)~~

   **_Upgrade-2019-11-15:_**

   1. 重装之后在 grub 界面，按 e 编辑启动项，在倒数第二行 quiet 后添加 `acpi_osi=! acpi_osi='Windows 2009'`
   2. 进入系统后，编辑 `/boot/grub/grub.cfg`，按 1 编辑该文件
   3. 进入 Manjaro Settings Manager，可以通过 mod+f3 -> Applications -> Preferences 中快捷方式打开或通过命令行输入 `manjaro-settings-manager` 打开。点击 Hardware Configuration -> Auto Install proprietary Driver 自动安装显卡驱动

3. 安装系统，注意安装的位置，是否分区，然后正常进入桌面环境

## 启动之后

1. 配置源

   ```sh
   sudo pacman-mirrors -i -c China -m rank

   # {-S --sync} 表示同步，pacman -S [options] <package(s)> 表示安装某包
   # options:
   #   {-y --refresh} 相当于刷新远程数据库，-yy 强制刷新
   #   {-u --sysupgrade} 更新本地安装的包
   #   {-s --search} 搜索
   sudo pacman -Syy
   ```

2. 使用 yay

   除了安装官方库中提供的包，也可以通过 yay 工具安装 AUR 包，基本使用命令同 pacman

   _使用中发现 pacman 查询包速度特别快，但是 yay 会慢一些，偶尔因网络原因还会卡住，可能是因为要额外搜索 AUR 仓库的原因(?)_

3. 安装 Chrome

   官方仓库中只有 Chromium，AUR 提供了 Chrome，自行选择

   ```sh
   # 这里可以省略 -S
   # 这里安装的是 stable 版，也有 beta 和 dev 版
   # 启动命令 google-chrome-stable
   yay google-chrome
   ```

4. 无法正常显示汉字

   由于安装的英文系统，系统某些地方无法正常显示汉字，比如 Chromium 的标题栏

   Q: 系统缺失字体

   A: 安装字体后 **重启** 生效

   ```sh
   sudo yay noto-fonts noto-fonts-cjk noto-fonts-emoji wqy-microhei wqy-miceohei-lite
   ```

5. 安装中文输入法

   ~~选择 fcitx 输入法框架~~

   ```sh
   # sudo pacman -S fcitx-gtk2 fcitx-gtk3 fcitx-qt4 fcitx-qt5
   sudo pacman -S fcitx-im
   sudo pacman -S fcitx-configtool
   sudo pacman -S fcitx-googlepinyin
   ```

   ~~修改配置文件 `~/.xprofile`，添加~~

   ```
   export GTK_IM_MODULE=fcitx
   export QT_IM_MODILE=fcitx
   export XMODIFUERS="@im=fcitx"
   ```

   ~~TODO: 每次重启之后都要输入 `fcitx` 手动启动(?)~~

   **_Upgrade-2019-11-15:_**

   选择 ibus 框架

   ```sh
   # ibus 为输入法框架，ibus-qt 是让输入法支持 qt 程序的包，ibus-rime 为具体的输入法
   yay ibus ibus-qt ibus-rime
   ```

   启动 ibus

   ```sh
   # 手动启动
   ibus-setup
   ```

   修改配置文件 `~/.xprofile`，添加

   ```sh
   export GTK_IM_MODULE=ibus
   export XMODIFIERS=@im=ibus
   export QT_IM_MODULE=ibus
   # 自动启动 ibus 进程
   ibus-daemon -d -x
   ```

   可手动启用该文件

   ```sh
   source ~/.xprofile
   ```

   在 ibus 配置中添加 rime 输入法。切换到该输入法，以下是部分配置方法

   1. rime 输入法默认是繁体字，打字时 ctrl+` 切换输入方式
   2. 在 `~/.config/ibus/rime` 文件夹新建 `default-custom.yaml` 文件

      ```yaml
      patch:
        # 每页选词数量
        'menu/page_size': 7
      ```

6. 修改默认浏览器

   修改 `～/.profile`

   ```sh
   export BROWSER=/usr/bin/google-chrome-stable
   ```

   修改 `~/.config/mimeapps.list`，把 `[Default Applications]` 下的 `userapp-Pale Moon.desktop` 修改为 `userapp-google-chrome-stable.desktop`，比如

   ```
   [Default Applications]
   x-scheme-handler/http=userapp-google-chrome-stable.desktop
   x-scheme-handler/https=userapp-google-chrome-stable.desktop
   x-scheme-handler/ftp=userapp-google-chrome-stable.desktop
   x-scheme-handler/chrome=userapp-google-chrome-stable.desktop
   ```

   _浏览器依然会询问是否设置为默认浏览器(?)_

7. 安装字体文件

   把字体文件放到 `/usr/share/fonts` 或 `~/.local/share/fonts` 文件夹，然后 `fc-cache -f`

8. 通过 CLI 访问 U 盘

   ```sh
   # 通过如下 3 个命令可以查看当前连接的硬盘和 U 盘，通常目标路径大概长这样 `/dev/sdb`
   lsblk
   blkid
   fdisk -l

   # 创建目录用来加载目标 U 盘，也可以是其它位置
   mkdir ~/media/usb

   # 加载 U 盘
   sudo mount /dev/sdb ~/media/usb

   # 取消加载
   sudo umount ~/media/usb
   ```

9. Timeshift

## i3wm

1. 常用快捷键

   ```sh
   # 默认 mod 键为 Super (Win)，可以修改为 Alt

   # 查看使用手册
   mod+shift+h

   # 打开 Terminal
   mod+enter
   # 切换工作区
   mod+1-8
   # 锁屏
   mod+9
   # 退出以及其它功能
   mod+0
   # 关闭当前窗口
   mod+shift+q
   # 切换工作区中的焦点
   mod+方向键或j,k,l,;
   # 切换工作区中焦点的位置
   mod+shift+方向键或j,k,l,;
   # 切换工作区
   mod+ctrl+方向键
   # 移动当前窗口到某工作区
   mod+shift+1-8
   # 移动当前整个工作区到某工作区
   mod+ctrl+1-8
   # 返回之前的工作区，重复按就是在两个工作区来回切换
   mod+b
   # 移动当前窗口到之前的工作区
   mod+shift+b
   # 默认打开的新窗口在水平方向，可以通过以下命令切换
   mod+h # 水平
   mod+v # 垂直
   mod+q # 在水平方向和垂直方向循环切换，窗口边框会有高亮提示（大概 1-2px 宽的白边吧 233）

   # 切换窗口为平铺模式（默认）
   mod+e
   # 切换窗口为 tab 模式（类似浏览器）
   mod+w
   # 切换窗口为堆叠模式（类似 Windows，但每个窗口都是最大化的）
   mod+s

   # 切换当前焦点为浮动模式（类似 Windows，但窗口位于桌面正中，不占满屏幕）
   # mod+鼠标拖动
   mod+shift+space
   # 在普通模式的窗口和浮动模式的窗口之前切换
   mod+space

   # 显示窗口边框为 1px（默认）
   mod+y
   # 不显示窗口边框
   mod+u
   # 显示窗口边框为普通（类似窗口显示标题栏，tab 模式和堆叠模式会强制显示，且不能切换）
   mod+n

   # 打开默认浏览器
   mod+f2
   # 打开默认资源管理器，默认为 pamanfm
   mod+f3

   # 打开 dMenu（桌面顶部一个提示输入和运行命令行的工具）
   mod+d
   # 打开 bMenu（一个在 Terminal 中操作的系统管理器）
   mod+ctrl+b
   # 打开 morc_menu
   mod+z

   # 重新载入 i3wm 配置文件
   mod+shift+c
   # 重启 i3wm
   mod+shift+r
   # 退出 i3wm，桌面顶部会有提示
   mod+shift+e

   # 关闭所有的 cli
   mod+ctrl+x
   ```

2. 修改过的地方，位置 `~/.i3/config`

   **_Upgrade-2019-11-15:_**

   1. 重装之后 dmenu 可以使用，无需修改
   2. `bindsym $mod+x exec st` 为 Simple Terminal 设置快捷键

   ```sh
   # 1. dmenu 无法打开
   #### 默认设置无效，但可以通过 cli 中输入 `dmenu_run` 启动（如果 cli 也无法启动，应该是 locale 设置不对）
   bindsym $mod+d exec --no-startup-id dmenu_recency
   # =>
   bindsym $mod+d exec --no-startup-id dmenu_run

   # 2. 修改默认打开的 terminal
   bindsym $mod+Return exec terminal
   # =>
   bindsym $mod+Return exec st

   # 3. 设置打开 Chrome 的快捷键
   bindsym $mod+c exec google-chrome-stable
   ```

## Terminal

1. 安装 Simple Terminal

   ```sh
   sudo pacman -S st-manjaro
   ```

2. 安装 oh-my-zsh （系统自带了 zsh）

   ```sh
   sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
   ```

3. 设置 zsh，配置文件在 `~/.zshrc`

   ```sh
   ZSH_THEME="agnoster"
   plugins=(
      git
      npm
      node
      vscode
      ng
      web-search
      # z
      # command-time
      # zsh-autosuggestions
      # zsh-syntax-highlighting
   )

   ### Add aliases in `~/.bash_aliases` to zsh
   source ~/.bash_aliases

   ### Add default user to hide user@hostname when using agnoster theme
   DEFAULT_USER=$USER

   ### Add nvm to zsh
   source /usr/share/nvm/init-nvm.sh

   ### command-time plugin config
   # ZSH_COMMAND_TIME_MIN_SECONDS=3
   # ZSH_COMMAND_TIME_MSG="Took: %s"
   # ZSH_COMMAND_TIME_COLOR="cyan"
   ```

4. 为了显示 zsh 某些主题，需要安装 Powerline fonts

   ```sh
   # clone
   git clone https://github.com/powerline/fonts.git --depth=1
   # install
   cd fonts
   ./install.sh
   # clean-up a bit
   cd ..
   rm -rf fonts
   ```

## Code

1. 安装 VScode，系统中叫 `Code - OSS` (why?)

   ```sh
   sudo pacman -S code
   ```

   启动命令：`code`

2. VScode 同步设置

   问题：使用 VScode 插件 Settings Sync 同步时登录失败，提示 `Client network socket disconnected before secure TLS connection was established`

   原因：可能是之前设置了 proxy 引起的

   关闭 VScode，CLI 中输入 `echo $http_proxy; echo $https_proxy`，如果显示 `127.0.0.1:1080`，修改变量的值为 `""`。启动一个新的 VScode，重新登录 Github

3. 安装 `node` 后再安装 `npm`

   ```sh
   sudo pacman -S nodejs
   sudo pacman -S npm
   ```

4. 或者选择使用 `nvm`

   ```sh
   yay nvm
   # 安装 lts 版本的 node 以及对应版本的 npm
   nvm install --lts
   ```

## SSR

- 如何在服务器上设置，查看 [wiki](https://github.com/Alvin9999/new-pac/wiki)

- ~~在 Linux 上启动 ssr, 复制该 [链接](https://github.com/the0demiurge/CharlesScripts/blob/master/charles/bin/ssr) 中的文件到 `/usr/local/bin`~~

  ~~为了让该脚本可运行，`chmod +x /usr/local/bin/ssr`~~

  ```sh
  # 查看帮助
  ssr help

  # 首次安装
  ssr install

  # 配置
  ssr config
  ```

  ~~编辑 `config.json` 文件后依然可运行，但没有效果，原因未知~~

- 改为使用 `shadowsocks-qt5` 客户端

  ```
  sudo pacman -S shadowsocks-qt5
  ```

  启动命令 `ss-qt5`

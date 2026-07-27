+++
title = "Fedora系统的安装和使用"
author = ["zbliang"]
date = 2026-07-24
tags = ["Fedora"]
categories = ["tech"]
draft = false
+++

笔记本是华为 `Matebook 16` 系列（型号 `HKFG-16` ），原本安装的系统是 `Ubuntu 24` 。
笔记本除了系统盘，还单独分出了一个名为 `data` 的分区，用作存档资料。
我们将把系统换成 `Fedora 44` 。
安装系统时要注意不要把 `data` 分区格式化。


## 选择英文为系统语言 {#选择英文为系统语言}

如果在安装系统时直接选择 `中文（简体）` ， `Fedora` 有可能会默认将用户主目录下的常用文件夹命名为中文：

-   `~/桌面` (Desktop)
-   `~/下载` (Downloads)
-   `~/文档` (Documents)
-   `~/图片` (Pictures)

那么以后在终端（ `Terminal` ）中操作时，需要频繁切换输入法输入中文，或者按下 `Tab` 键补全时容易出现字符编码或路径转义的问题，体验非常糟心。

所以推荐选英文（ `English (United States)` ）作为系统语言，这样所有默认文件夹名均为英文（ `Desktop` ， `Downloads` ， `Documents` ），终端操作流畅。
如果之后想换成中文，进入系统后，可以随时在 `Settings -> Region & Language` 中将语言切换为中文，或者仅保留系统菜单为中文，同时保留英文的目录名。

如果不小心已经变成中文目录，也无需重新安装，在终端运行以下一行命令即可自动改回英文：

```text
LANG=en_US xdg-user-dirs-gtk-update
```

运行后会弹出一个窗口，提示将中文目录重命名为英文目录（如 `桌面` -&gt; `Desktop` ），确认修改即可。


## 划分磁盘 {#划分磁盘}

当安装 `Fedora` 进行到 `How would you like to install?` 界面时，千万不能直接选择全盘自动安装（ `Automatic` ），因为那可能会把 `data` 盘一并格式化掉。
要选择 `Custom` （自定义）或 `Advanced Custom` （Blivet-GUI）。

这时会出现 `Mount point assignment` ，要求 `select a device` ，有四个选项：

-   `nvme0n1p1 1.13 GB EFI System Partition`
-   `nvme0n1p2 2.15 GB ext4`
-   `ubuntu-vg-data 215 GB ext4`
-   `ubuntu-va-ubuntu-lv 789 GB ext4`

这提示之前在 `Ubuntu` 下使用的是 `LVM` （逻辑卷管理） 架构。
从列表的大小和名称来看，各个分区的角色分别是：

-   `nvme0n1p1` (1.13 GB)： `EFI` 引导分区。
-   `nvme0n1p2` (2.15 GB)：以前 `Ubuntu` 的 `/boot` 引导分区。
-   `ubuntu-vg-data` (215 GB)：这就是不能格式化的 `data` 资料盘！
-   `ubuntu-va-ubuntu-lv` (789 GB)：以前 `Ubuntu` 的主系统盘（根目录 `/` ）

各部分按以下参数进行选择：

1.  `Mount point: / (required)`  —— 【Fedora 的根系统盘】
    -   `Select a device` 选择 `ubuntu-va-ubuntu-lv (789 GB)`
    -   `Reformat ?` 勾选 `Yes` （必须格式化，这样会把旧的 `Ubuntu` 系统完全清除，替换为新的 `Fedora` 系统。）
2.  `/boot/efi (required)` —— 【EFI 启动引导区】
    -   `Select a device` 选择 `nvme0n1p1 (1.13GB EFI System Partition)`
    -   `Reformat ?` 勾选 `No` （千万不要格式化，直接复用原有的 EFI 引导分区。）
3.  `/boot (Recommended)` —— 【Linux 内核引导区】
    -   `Select a device` 选择 `nvme0n1p2 (2.15 GB ext4)`
    -   `Reformat ? 勾选 =Yes` （建议格式化，清理掉旧 Ubuntu 的内核文件，让 Fedora 重新写入它的引导文件。)
4.  添加自定义挂载点，如 `+` 号或添加挂载点
    -   选中 `ubuntu-vg-data (215 GB)`
    -   `Mount point` 可以设置为 `/data`
    -   `Reformat ?` 绝不能勾选！

这样做， `/data` 将作为根目录下的独立文件夹。
在安装完成后，只需执行一次权限修改命令（比如 `sudo chown -R $USER:$USER /data` ），就可以像使用普通文件夹一样随时读写 `/data` 文件夹，非常适合作为个人的通用资料库（代码项目、下载文件、大容量文档等）。

在这台华为笔记本（搭载 `NVMe` 高速固态硬盘）上 ， `Fedora`  的安装速度非常快，大约只需 3 到 7 分钟。


## 系统安装后的简单设置 {#系统安装后的简单设置}

当询问是否 `Enable Third-Party Repositories` 时，直接勾选。
在开源操作系统中，由于版权和许可协议的限制， `Fedora` 默认的官方软件库只包含纯粹的「自由/开源软件」。
如果不开启第三方软件库（ `Third-Party Repositories` ），在后续使用中可能会十分不方便。
开启后， `Fedora` 软件中心（ `Software` ）会自动集成 `Flathub` 和 `RPM Fusion` ，方便用户下载许多日常软件。
而且，这不会对系统安全性产生威胁的。

平时登录系统用的用户密码可以设置得简单一点（比如 4-6 位）。
以后经常使用 `sudo` 提权安装软件、运行命令，需要输入用户密码，如果密码简单，输入就方便。

使用终端完成初次系统更新，在终端执行

```text
sudo dnf update -y
```

这时终端会开始拉取软件源列表，并下载最新补丁。
遇到问 `Is this ok [y/N]:` 的地方，系统会自动选 `y` （因为上面命令加入了参数 `-y` ），耐心等待跑完即可。

如果以后觉得下载更新速度太慢，可以给 `Fedora` 的软件源开启自动选择最快镜像。
在终端运行以下命令修改配置

```text
echo "fastestmirror=True" | sudo tee -a /etc/dnf/dnf.conf
echo "max_parallel_downloads=10" | sudo tee -a /etc/dnf/dnf.conf
```


## 解决扬声器不发声的问题 {#解决扬声器不发声的问题}

部分华为 `MateBook 16` 系列在 `Linux` （包括 `Fedora` 、 `Ubuntu` ）下默认没有声音，这是因为其音频芯片（如 `Conexant CX8070/CX11880` 或 `Cirrus Logic/ES8336` ）采用了一些非标准的硬件映射和 `GPIO` 控制逻辑。
`Linux` 内核的默认声卡驱动无法正确切换揚声器的通道，导致系统显示有音频播放，但扬声器不发声。
不过这只是硬件兼容性/软件配置问题，而不是硬件故障。

社区开发者针对华为 `MateBook 14s / 16 / 16s` 等机型的 `soundcard bug` 编写了修复工具。
该工具通过监控音频状态并发送对应的 `GPIO / EAPD` 信号来正确唤醒扬声器。

打开终端，安装必要的构建与音频工具

```text
sudo dnf install -y git alsa-tools alsa-utils
```

克隆修复仓库并运行

```text
git clone https://github.com/Smoren/huawei-ubuntu-sound-fix.git
cd huawei-ubuntu-sound-fix
sudo bash install.sh
```

检查服务状态或重启音频服务

```text
sudo systemctl status huawei-soundcard-headphones-monitor
```


## 安装中文输入法 {#安装中文输入法}

在 `Fedora` 中（默认采用 `GNOME` 桌面环境），系统已经预装并内置了对 `IBus` 智能拼音（ `Intelligent Pinyin` ）的支持。
用户不需要安装额外的软件，只需要在系统设置中把它激活出来即可。

1.  打开系统的 `Settings` （设置）。
2.  在左侧菜单找到 `Keyboard` （键盘）。
3.  在右侧的 `Input Sources` （输入源） 区域，点击 `Add Source...` （添加输入源 `+` 号）。
4.  在弹出的窗口中：
    -   点击三点图标或搜索框，找到并选择 `Chinese (China)` / `中文（中国）` 。
    -   选择 `Chinese (Intelligent Pinyin)` / `中文（智能拼音）` 。
    -   点击右上角的 `Add` （添加）。

添加后，默认通过按 `Super + Space` （ `Windows 键 + 空格键` ） 或 `Shift` 键在中英文之间切换。
顶部状态栏右上角会出现拼音图标，点击可进入设置（设置词库、双拼模式等）。


## 调整 `GNOME` 桌面环境 {#调整-gnome-桌面环境}


### 找回窗口的「最小化」和「最大化」按钮 {#找回窗口的-最小化-和-最大化-按钮}

在 `Fedora` 中打开软件，软件的窗口并没有没有「最小化」和「最大化」按钮。
要找回这两个按钮，可以通过系统自带的 `Tweaks` （优化工具） 开启：

安装 `GNOME Tweaks` （如果系统没预装）。
打开终端运行：

```text
sudo dnf install -y gnome-tweaks
```

-   然后按键盘 `Super` 键（ `Windows` 徽标键），搜索并打开 `Tweaks` （优化）。
-   在左侧菜单中点击 `Windows` （窗口） 或 `Window Titlebars` （窗口标题栏）。
-   找到 `Titlebar Buttons` （标题栏按钮） 区域，把 `Minimize` （最小化） 和 `Maximize` （最大化） 的开关都勾选上。


### 开启桌面右键菜单 {#开启桌面右键菜单}

`GNOME` 本身不支持桌面图标，所以无法支持在桌面新建文件夹，以及放图标（尽管具有 `~/ Desktop` 文件夹）。
可以通过安装官方扩展 `Ding` （ `Desktop Icons Neo GNOME` ）来恢复传统桌面的功能：

-   终端安装扩展包
    ```text
    sudo dnf install -y gnome-shell-extension-desktop-icons-gnome
    ```
-   启用扩展
    -   按 `Super` 键，搜索并打开 `Extensions` （扩展） 应用。
    -   在列表中找到 `Desktop Icons Neo` 或 `Desktop Icons` ，将右侧的开关打开为 `ON` 。

启用后，回到桌面鼠标右键，就能看到经典的「新建文件夹」、「新建文档」 等菜单。


### 开启桌面右上角系统托盘 {#开启桌面右上角系统托盘}

`GNOME` 默认禁用了顶栏的系统托盘（ `App Indicators / System Tray` ）功能。
可以通过 `Extensions` （ `扩展管理应用` ） 开启这个功能。

首先在系统自带的 `Software` （软件中心） 里看到搜索 `Extensions` （图标通常是一个带有绿色 puzzle 拼图形状的应用）。
找到后点击安装（ `Install` ）。

也可以在终端安装

```text
sudo dnf install -y gnome-extensions-app
```

安装完成后，按键盘 `Super` 键（ `Windows` 徽标键），搜索 `Extensions` 并打开。
在列表中找到 `AppIndicator and KStatusNotifierItem Support` ，将它右侧的开关切换为 `ON` 。

此时把 `Nextcloud` 关闭重新打开，桌面右上角就会立刻出现对应的小托盘图标。


### 设置像 macOS 那样「智能隐藏/自动悬浮」的程序坞 {#设置像-macos-那样-智能隐藏-自动悬浮-的程序坞}

在原生 `Fedora` （ `GNOME` 桌面环境），要令程序坞常驻在桌面底部/侧边，且有窗口挡住时自动隐藏、鼠标贴到边框时自动弹起，需要安装 `Linux` 界著名的扩展 `Dash to Dock` 。

首先安装扩展管理器 `Extension Manager` ，在终端运行

```text
sudo dnf install -y gnome-shell-extension-installer gnome-extensions-app
```

也可以直接在软件中心 `Software` 搜索安装 `Extension Manager` 。

-   安装后打开扩展管理器 `Extension Manager` 。
-   切换到 `Browse` 选项卡，搜索 `Dash to Dock` ，点击安装 `Install` 。

安装后，程序坞就会立刻显示在桌面底部。
点击 `Dash to Dock` 旁边的齿轮按钮进行设置。
找到 `Intelligent autohide` （智能自动隐藏） 开关并开启，并点击该选项旁边的齿轮，进一步设置：

-   `Autohide` ，勾选 `Push to show: require pressure to show the dock` 以及 `Show dock for urgent notifications` ，其他不勾选。
-   `Dodge windows` ，勾选 `All windows` ，其他不勾选。

此时，窗口最大化或覆盖到程序坞时，程序坞会自动隐藏，但鼠标移动到屏幕最边缘时，程序坞会弹起。
当窗口离开程序坞时，程序坞恢复显示。


### 调整系统字体大小 {#调整系统字体大小}

在 `Fedora` （ `GNOME` 桌面）上调整系统字体大小，可以通过 `GNOME` 官方的辅助配置工具 `Tweaks`  （优化）。

-   按键盘 `Super` 键（ `Windows` 键），搜索并打开 `Tweaks` （优化）。
    如果没有，可以在终端安装 `sudo dnf install -y gnome-tweaks` 。
-   在左侧菜单点击「字体」 `Fonts` ，找到缩放比例 `Scaling Factor` 。
    默认只是 `1.00` ，可以调整为比如 `1.25` ，那么全局字号会按比例平滑放大。


## 更新软件 {#更新软件}

基础日常更新可以通过在终端执行

```text
sudo dnf upgrade --refresh
```

其中

-   `upgrade` 表示检查并下载所有有新版本的软件包和系统补丁进行升级。
-   `--refresh` 表示强制刷一下最新的软件源缓存，确保获取到的是实时发布的更新。

当终端提示是否继续时，按下 `y` 回车即可。

定期清理无用缓存（释放磁盘空间）：

```text
sudo dnf clean all
```

以及清理孤立的无用依赖包：

```text
sudo dnf autoremove
```

更新完成后如果看到输出了新的 `kernel` 版本，建议重启一次电脑，系统就会自动加载最新的 `Linux` 内核运行：

```text
reboot
```

如果通过 `Flatpak` 安装了 `Zotero` 、 `VS Code` 或其他应用，它们不属于 `dnf` 管理范畴，需要单独运行一条命令来更新：

```text
flatpak update
```


## 版本升级 {#版本升级}

在 `Fedora` 中，「升级」通常指跨版本的大版本升级（例如从当前使用的 `Fedora 43` 或更早版本，升到 `Fedora 44` ）。
`Fedora` 的版本生命周期模型非常规律且清晰：

-   发布周期：大约每 6 个月发布一个新版本（通常在每年的 4 月底和 10 月底）。
-   支持周期：每个大版本支持大约 13 个月（即下一个版本发布后再维护 1 个月，或者下下个版本发布后维护 4 周）。

建议在新版本发布后的 1 到 2 个月 再升级。
这是因为，虽然新版本发布前经过大量测试，但刚发行的前几周往往是第三方扩展（如 `GNOME` 插件、 `Nvidia` 显卡驱动等）兼容性问题的集中解决期。
所以等 1~2 个月后升级，体验最为平滑。

`Fedora` 也允许「隔代升级」：由于支持周期覆盖 13 个月，完全可以跳过一个版本（例如从 `Fedora 42` 直接升到 `Fedora 44` ，不必先升到 `43` ）。

强制升级界限：在当前版本的 `EOL` （生命周期结束）前必须升级。
一旦 `EOL` ，该版本将不再接收任何安全补丁和软件更新。

跨大版本升级共有两种官方推荐方式。

方案 1：使用 `dnf system-upgrade` 命令行。

这种方式最稳当，能清楚看到所有依赖包的变化，是 Linux 熟练用户的首选。

更新前的准备工作：

-   确保所有个人重要文件已备份。
-   打开终端，先把当前版本的所有软件升到最新： `sudo dnf upgrade --refresh -y` 。
-   完成后重启一次电脑，确保系统运行在最新的内核下。

然后，安装升级插件并下载新版软件包

-   安装 DNF 升级插件（通常已内置）：
    ```text
    sudo dnf install dnf-plugin-system-upgrade -y
    ```
-   然后下载目标版本的升级包（以升级到 Fedora 44 为例）：
    ```text
    sudo dnf system-upgrade download --releasever=44
    ```
    如果在下载过程中遇到软件依赖冲突，可以追加 `--allowerasing` 参数让 `DNF` 自动移除旧版不兼容的依赖 `package`
-   当所有软件包下载完毕后，运行以下命令重启系统并进入离线升级阶段：
    ```text
    sudo dnf system-upgrade reboot
    ```

电脑会自动重启，并在开机界面显示「正在升级系统」进度条（耗时约 10~30 分钟，取决于网速和硬件性能）。
在此期间切勿断电。

方案 2：通过 `GNOME Software` 图形界面（傻瓜式点按）。

打开软件中心 `Software` ，进入更新 `Updates` 选项卡，点击左上角的刷新。

当系统检测到新版本发布时，顶部会弹出醒目的提示栏「Fedora 44 现在可用」。
点击下载 `Download` 按钮。
下载完成后，点击重启并升级 `Restart & Upgrade` 即可。

升级完成后的可选清理步骤升级进入新系统后，打开终端运行以下命令清理旧系统遗留的无用缓存与孤立包：

```text
# 清理系统旧的 DNF 缓存
sudo dnf system-upgrade clean

# 查找并移除不再被任何软件依赖的废弃旧软件包
sudo dnf autoremove
```


## 备份策略 {#备份策略}

跨版本升级系统虽然非常稳定，但还是有必要进行备份以防万一（如断电、磁盘故障或配置冲突）。

升级前的备份原则很简单：不备份系统软件本身（因为升级会全部重写），只备份你的 ****个人数据**** 和 ****个性化配置**** 。

需要备份的内容主要分为三大类：

第一类是个人核心数据，通常是用户主目录（ `/home/用户名/` ）下的个人文件，包括

-   `文档/图片/下载/视频` （ `Documents` ， `Pictures` ， `Downloads` 等）
-   工作项目/代码（如你的 `Git` 项目、 `LaTeX` 源文件等）

第二类是软件配置与应用数据。

一般来说，配置文件是隐藏在 `/home/用户名/` 路径下，按 `Ctrl + H` 可显形。
比如：

-   `.config/` 存放了大部分软件的个性化配置（如 `VS Code` 插件设置、输入法字典、快捷键等）。
-   `.ssh/` 保存了 SSH 密钥对（连接远程服务器必备）。
-   `.zotero/` 或 `Zotero` 数据目录，保存了 `Zotero` 数据库与本地 `PDF` 缓存。

如果 `Chrome/Firefox` 开启了账号同步，则无需手动备份；未同步建议导出书签。

第三类是已安装的软件清单。

记录下当前安装过哪些软件，万一需要重装，可以一键恢复：

```text
# 导出自己通过 DNF 手动安装的软件列表
dnf history userinstalled > ~/desktop-packages.txt

# 导出已安装的 Flatpak 应用列表
flatpak list --app --columns=application > ~/flatpak-apps.txt
```

对于绝大多数日常使用，只需把 `/home/你的用户名/` 下的个人文档、隐藏文件夹（如 `.config` 、 `.ssh` ）、以及软件列表，直接「手动」拷贝到移动硬盘或网盘中，备份工作就完成了！
完成这一步后，你就可以放心地去执行 `dnf system-upgrade` 进行系统大版本升级了。

如果熟悉命令行，推荐用 `rsync` 备份。
这样做速度最快且能保留文件权限属性。

先通过在终端运行以下命令查看是否已安装

```text
rsync --version
```

如果没有安装，那么通过 `dnf` 安装

```text
sudo dnf install -y rsync
```

安装后就可以备份了：

```text
# 假设你的移动硬盘挂载在 /run/media/你的用户名/BackupDrive
rsync -avh --progress --exclude='Downloads' --exclude='.cache' /home/你的用户名/ /run/media/你的用户名/BackupDrive/fedora_home_backup/
```

其中

-   `--exclude='Downloads'` 或 `--exclude='/Downloads'` 表示跳过了没必要备份的下载文件夹
-   `--exclude='.cache'` 表示跳过了系统的临时缓存 `.cache` 。
-   如果要一次排除多个文件，可以
    ```text
    rsync -avh --progress --exclude={'Downloads','.cache','.local/share/Trash'} /home/你的用户名/ /run/media/你的用户名/BackupDrive/fedora_home_backup/
    ```
-   `--progress` 表示在终端界面中显示实时进度条。
-   `-r` （Archive，归档模式）是一个快捷组合参数，相当于同时开启了 `-rlptgoD` 多个选项。
    它的核心作用是「原汁原味地精准复制」，包含：

    -   递归复制 `-r` ：自动复制文件夹及其子文件夹里的所有内容。
    -   保持属性：保留文件的修改时间 `-t` 、权限 `-p` 、所有者 `-o` 和用户组 `-g` 。
    -   保持链接：保留软链接/符号链接 `-l` 。

    如果不用 `-a` ，复制过去的文件所有修改时间都会变成「今天/现在」，文件的系统权限也可能会混乱。
-   `-v` （Verbose，详细模式）让 `rsync` 在终端里输出「正在传输的文件列表」。
-   `-h` （Human-readable，人类可读格式）将传输中的文件大小和传输速度转换为人们易读的单位。

在 `rsync` 命令中， `-avh --progress` 是日常用来做文件备份时最标准、最实用的组合参数。

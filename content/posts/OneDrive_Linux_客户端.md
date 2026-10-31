+++
title = "Linux 安装和使用 OneDrive 客户端"
author = ["zbliang"]
date = 2026-10-31
tags = ["Linux", "OneDrive"]
categories = ["tech"]
draft = false
+++

## 环境说明 {#环境说明}

本文记录在 Fedora 44 上安装和使用开源 OneDrive Linux 客户端的方法。

-   操作系统：Fedora 44
-   客户端：abraunegg/onedrive
-   已测试版本：2.5.11
-   OneDrive 类型：个人 Microsoft 账户
-   默认本地同步目录：~/OneDrive
-   默认配置目录：~/.config/onedrive

该客户端支持双向同步：

-   云端新增或修改文件，可以下载到本地。
-   本地新增或修改文件，可以上传到云端。
-   本地或云端删除文件，也可能传播到另一端。

注意：双向同步不是备份。
如果误删文件，删除操作可能同步到另一端。


## 安装 OneDrive 客户端 {#安装-onedrive-客户端}


### 检查 Fedora 版本 {#检查-fedora-版本}

```bash
cat /etc/fedora-release
```

```text
Fedora release 44 (Forty Four)
```


### 检查软件包信息 {#检查软件包信息}

```bash
dnf info onedrive
```

```text
Repositories loaded.
Available packages
Name           : onedrive
Epoch          : 0
Version        : 2.5.11
Release        : 1.fc44
Architecture   : x86_64
Download size  : 1.1 MiB
Installed size : 4.1 MiB
Source         : onedrive-2.5.11-1.fc44.src.rpm
Repository     : updates
Summary        : OneDrive Free Client written in D
URL            : https://github.com/abraunegg/onedrive
License        : GPL-3.0-only
Description    : Free CLI client for Microsoft OneDrive written in D.
Vendor         : Fedora Project

```

Fedora 44 软件仓库中可以安装 abraunegg/onedrive。


### 安装 {#安装}

先检查是否安装过 onedrive：

```bash
rpm -q onedrive
```

```text
package onedrive is not installed
```

安装 onedrive：

```bash
sudo dnf install onedrive
```


### 检查安装结果 {#检查安装结果}

```bash
onedrive --version
```

```text
onedrive v2.5.11
```

查看当前版本支持的命令：

```bash
onedrive --help
```

```text
onedrive - A client for the Microsoft OneDrive Cloud Service

  Usage:
    onedrive [options] --sync
      Do a one-time synchronisation with Microsoft OneDrive
    onedrive [options] --monitor
      Monitor filesystem and synchronise regularly with Microsoft OneDrive
    onedrive [options] --display-config
      Display the currently used configuration
    onedrive [options] --display-sync-status
      Query OneDrive service and report on pending changes
    onedrive -h | --help
      Show this help screen
    onedrive --version
      Show version

  Options:

  --auth-files '<path or required value>'
      Perform authentication via files rather than an interactive dialogue. The application reads/writes the required values from/to the specified files
  --auth-response '<path or required value>'
      Perform authentication via a supplied response URL rather than an interactive dialogue
  --check-for-nomount
      Check for the presence of .nosync in the syncdir root. If found, do not perform sync
  --check-for-nosync
      Check for the presence of .nosync in each directory. If found, skip directory from sync
  --classify-as-big-delete '<path or required value>'
      Number of children in a path that is locally removed which will be classified as a 'big data delete'
  --cleanup-local-files
      Clean up additional local files when using --download-only. This will remove local data
  --confdir '<path or required value>'
      Set the directory used to store the configuration files
  --create-directory '<path or required value>'
      Create a directory on OneDrive. No synchronisation will be performed
  --create-share-link '<path or required value>'
      Create a shareable link for an existing file on OneDrive
  --debug-https
      Debug OneDrive HTTPS communication.
  --destination-directory '<path or required value>'
      Destination directory for renamed or moved items on OneDrive. No synchronisation will be performed
  --disable-download-validation
      Disable download validation when downloading from OneDrive
  --disable-notifications
      Do not use desktop notifications in monitor mode
  --disable-upload-validation
      Disable upload validation when uploading to OneDrive
  --display-admin-consent-url
      Displays a tenant-specific Microsoft Entra ID administrator consent URL for environments where administrator approval is required before users can authenticate and use the client
  --display-config
      Display what options the client will use as currently configured. No synchronisation will be performed
  --display-quota
      Display the quota status of the client. No synchronisation will be performed
  --display-running-config
      Display what options the client has been configured to use on application startup
  --display-sync-status
      Display the sync status of the client. No synchronisation will be performed
  --download-file '<path or required value>'
      Download a single file from Microsoft OneDrive
  --download-only
      Replicate the OneDrive online state locally, by only downloading changes from OneDrive. Do not upload local changes to OneDrive
  --dry-run
      Perform a trial sync with no changes made
  --enable-logging
      Enable client activity to a separate log file
  --file-fragment-size
      Specify the file fragment size for large file uploads (in MB)
  --force
      Force the deletion of data when a 'big delete' is detected
  --force-http-11
      Force the use of HTTP 1.1 for all operations
  --force-sync
      Force a synchronisation of a specific folder, only when using --sync --single-directory and ignore all non-default skip_dir and skip_file rules
  --get-file-link '<path or required value>'
      Display the file link of a synced file
  --get-sharepoint-drive-id '<path or required value>'
      Query and return the Office 365 Drive ID for a given Office 365 SharePoint Shared Library
  --help -h
      This help information.
  --list-shared-items
      List OneDrive Business Shared Items
  --local-first
      Synchronise from the local directory source first, before downloading changes from OneDrive
  --log-dir '<path or required value>'
      Directory where logging output is saved to, needs to end with a slash
  --logout
      Log out the current user
  --mirror-local-state
      Mirror the local state to Microsoft OneDrive online
  --modified-by '<path or required value>'
      Display the last modified by details of a given path
  --monitor -m
      Keep monitoring for local and remote changes
  --monitor-fullscan-frequency '<path or required value>'
      Number of scheduled monitor-interval sync cycles before performing an online full-scan true-up. This does not disable the local database consistency check.
  --monitor-interval '<path or required value>'
      Number of seconds by which each sync operation is undertaken when idle under monitor mode
  --monitor-log-frequency '<path or required value>'
      Frequency of logging in monitor mode
  --no-remote-delete
      Do not delete local file 'deletes' from OneDrive when using --upload-only
  --print-access-token
      Print the access token, useful for debugging
  --reauth
      Reauthenticate the client with OneDrive
  --remove-directory '<path or required value>'
      Remove a directory on OneDrive. No synchronisation will be performed
  --remove-source-files
      Remove source file after successful transfer to OneDrive when using --upload-only
  --remove-source-folders
      Remove the local directory structure post successful file transfer to Microsoft OneDrive when using --upload-only --remove-source-files
  --resync
      Forget the last saved state, perform a full sync
  --resync-auth
      Approve the use of performing a --resync action
  --share-password '<path or required value>'
      Require a password to access the shared link when used with --create-share-link <file>
  --single-directory '<path or required value>'
      Specify a single local directory within the OneDrive root to sync
  --skip-dir '<path or required value>'
      Skip any directories that match this pattern from syncing
  --skip-dir-strict-match
      When matching skip_dir directories, only match explicit matches
  --skip-dot-files
      Skip dot files and folders from syncing
  --skip-file '<path or required value>'
      Skip any files that match this pattern from syncing
  --skip-size '<path or required value>'
      Skip new files larger than this size (in MB)
  --skip-symlinks
      Skip syncing of symlinks
  --source-directory '<path or required value>'
      Source directory to rename or move on OneDrive. No synchronisation will be performed
  --space-reservation '<path or required value>'
      The amount of disk space to reserve (in MB) to avoid 100% disk space utilisation
  --sync -s
      Perform a synchronisation with Microsoft OneDrive
  --sync-root-files
      Sync all files in sync_dir root when using sync_list
  --sync-shared-files
      Sync OneDrive Business Shared Files to the local filesystem
  --syncdir '<path or required value>'
      Specify the local directory used for synchronisation to OneDrive
  --threads
      Specify a value for the number of worker threads used for parallel upload and download operations
  --upload-only
      Replicate the locally configured sync_dir state to OneDrive, by only uploading local changes to OneDrive. Do not download changes from OneDrive
  --verbose -v+
      Print more details, useful for debugging (repeat for extra debugging)
  --version
      Print the version and exit
  --with-editing-perms
      Create a read-write shareable link for an existing file on OneDrive when used with --create-share-link <file>
```

注意：

-   正确的同步选项是 `--sync` 。
-   `--synchronize` 和 `--list` 不是本次安装版本支持的选项。


## 首次授权 Microsoft 账户 {#首次授权-microsoft-账户}


### 启动客户端 {#启动客户端}

在 GNOME 桌面环境下打开终端，执行：

```bash
onedrive
```


### 浏览器授权 {#浏览器授权}

客户端会自动打开默认浏览器，并跳转到 Microsoft 登录页面。

操作步骤：

1.  使用 Microsoft 账户登录。
2.  确认授权请求。
3.  等待浏览器完成授权。
4.  返回终端，检查是否出现授权成功提示。

成功时，终端会显示类似：

```text
The application has been successfully authorised,
but no extra command options have been specified.
```


### 检查授权文件 {#检查授权文件}

```bash
ls -lah ~/.config/onedrive
```

授权后通常可以看到：

-   `refresh_token` ：Microsoft 账户授权令牌。
-   `items.sqlite3` ：本地同步状态数据库。

注意：

-   不要删除 `refresh_token` ，否则可能需要重新授权。
-   不要把 `refresh_token` 的内容发送给他人。
-   不要随意删除 `items.sqlite3` 。


## 配置 OneDrive {#配置-onedrive}


### 查看默认配置 {#查看默认配置}

```bash
onedrive --display-config
```

初始配置通常包括：

-   配置目录： `~/.config/onedrive`
-   本地同步目录： `~/OneDrive`
-   `sync_list` ：未配置，即不限制同步目录范围。
-   `monitor_interval` ：300 秒。


### 创建配置文件 {#创建配置文件}

Fedora 软件包提供配置模板：

```bash
/usr/share/doc/onedrive/config
```

如果尚未创建配置文件，可以执行：

```bash
cp /usr/share/doc/onedrive/config ~/.config/onedrive/config
```

注意：如果已经存在配置文件，不要直接覆盖。


### 编辑配置文件 {#编辑配置文件}

```bash
nano ~/.config/onedrive/config
```

配置文件中的选项通常以 `#` 注释开头。
需要修改某个选项时，取消注释并修改对应的值。

本次采用的主要配置：

```cfg
use_recycle_bin = "true"
bypass_data_preservation = "false"
permanent_delete = "false"
```

这些选项的含义：

-   `use_recycle_bin = "true"` ：当云端文件被删除时，将其移入本地系统的回收站，以便检查和恢复。
-   `bypass_data_preservation = "false"` ：保留客户端的数据保护机制。
-   `permanent_delete = "false"` ：不启用永久删除。

特别注意：

`use_recycle_bin` 不是普通的本地删除保护，也不是备份机制。
它主要针对云端删除的文件在本地同步时的处理。


### 保持整个 OneDrive 云盘同步 {#保持整个-onedrive-云盘同步}

如果希望同步整个个人 OneDrive：

-   不要配置 `sync_list` 文件。
-   不要设置只同步某个目录的 `--single-directory` 。
-   不要启用 `download_only` 或 `upload_only` 作为长期运行模式。

默认同步目录：

```cfg
sync_dir = "~/OneDrive"
```


### 检查目前的配置 {#检查目前的配置}

```bash
onedrive --display-config
```

重点确认：

-   `sync_dir = ~/OneDrive`
-   `use_recycle_bin = true`
-   `bypass_data_preservation = false`
-   `permanent_delete = false`
-   `sync_list` 未配置


## 首次同步前的检查 {#首次同步前的检查}

首次同步前，建议依次完成以下检查。


### 检查云端容量 {#检查云端容量}

```bash
onedrive --display-quota
```

确认云端文件总量能够放进本地磁盘。


### 检查本地磁盘空间 {#检查本地磁盘空间}

```bash
df -h ~
```


### 检查本地同步目录 {#检查本地同步目录}

```bash
ls -lah ~/OneDrive
du -sh ~/OneDrive
```

如果目录已经存在，要先确认其中有没有重要文件。


### 执行模拟同步 {#执行模拟同步}

```bash
onedrive --sync --dry-run
```

该命令只模拟同步，不会真正下载、上传或删除文件。

重点检查：

-   是否成功连接 Microsoft OneDrive。
-   是否扫描了预期的本地同步目录。
-   是否出现意外的上传操作。
-   是否出现意外的删除操作。
-   是否出现文件冲突或其他错误。

注意：

模拟同步输出中的：

```text
No changes or items that can be applied
```

只表示本轮没有发现可以应用的变更，不能据此断定整个云盘的文件已经下载完成。


## 执行首次同步 {#执行首次同步}

根据实际需要选择同步方式。


### 推荐的谨慎方式：只下载云端文件 {#推荐的谨慎方式-只下载云端文件}

```bash
onedrive --sync --download-only
```

作用：

-   将云端文件下载到本地。
-   不上传本地新增或修改的文件。
-   不会因为云端删除而自动删除本地已有文件（2.5.x 版本默认行为）。

适合第一次下载整个云盘，先建立本地副本。


### 双向同步 {#双向同步}

```bash
onedrive --sync
```

作用：

-   下载云端新增或修改的文件。
-   上传本地新增或修改的文件。
-   按正常同步规则处理删除和冲突。

如果希望本地和云端保持双向同步，最终使用这种方式。

注意：不要在不了解影响的情况下使用以下选项：

```bash
onedrive --sync --upload-only
onedrive --sync --download-only --cleanup-local-files
onedrive --resync
```

其中：

-   `--upload-only` 可能把本地缺失的云端文件删除。
-   `--cleanup-local-files` 会清理本地文件。
-   `--resync` 会重新建立同步状态，可能重新识别本地文件并上传。


## 日常使用命令 {#日常使用命令}


### 手动执行一次双向同步 {#手动执行一次双向同步}

```bash
onedrive --sync
```

也可以简单输入

```bash
onedrive -s
```

要注意的是 `onedrive -s` 只执行一次同步，同步完成后退出程序。
如果要持续监控本地和云端变化，并在需要时自动同步，那么应该使用后面将要介绍的 `onedrive --monitor` （简写 `onedrive -m` ）。


### 手动执行一次只下载同步 {#手动执行一次只下载同步}

```bash
onedrive --sync --download-only
```


### 查看配置 {#查看配置}

```bash
onedrive --display-config
```


### 查看云端容量 {#查看云端容量}

```bash
onedrive --display-quota
```


### 查看同步状态 {#查看同步状态}

```bash
onedrive --display-sync-status
```


### 模拟同步 {#模拟同步}

```bash
onedrive --sync --dry-run
```


### 查看版本 {#查看版本}

```bash
onedrive --version
```


### 查看帮助 {#查看帮助}

```bash
onedrive --help
```


## 后台自动同步 {#后台自动同步}

客户端有两种常见运行方式。


### 前台监控模式 {#前台监控模式}

```bash
onedrive --monitor
```

也可以使用简写：

```bash
onedrive -m
```

客户端会持续监控本地和云端变化，并按需要执行同步。

退出监控模式可以按：

```text
Ctrl + C
```


### 使用 systemd 用户服务 {#使用-systemd-用户服务}

在已经完成首次授权并确认手动同步正常后，可以启用后台服务。

启动并设置登录后自动运行：

```bash
systemctl --user enable --now onedrive.service
```

查看服务状态：

```bash
systemctl --user status onedrive.service
```

查看实时日志：

```bash
journalctl --user -u onedrive.service -f
```

停止服务：

```bash
systemctl --user stop onedrive.service
```

禁止登录后自动启动：

```bash
systemctl --user disable onedrive.service
```

停止并取消自动启动：

```bash
systemctl --user disable --now onedrive.service
```

注意：

-   以上命令使用当前普通用户执行，不要加 sudo。
-   启用服务前，先确认首次手动同步已经正常完成。
-   如果 Fedora 软件包没有提供相应的用户服务单元，需要先检查服务单元是否存在。
-   后台服务启动后，不要再同时启动另一个独立的 `onedrive --monitor` 进程。


## 更换 Microsoft OneDrive 账户 {#更换-microsoft-onedrive-账户}

如果需要更换账户，可以采用两种方式。

方式一：重新授权

```bash
onedrive --reauth
```

该命令清理原授权并重新打开 Microsoft 登录授权流程。

如果正在运行后台服务，应先停止服务。

方式二：彻底清除旧账户的本地数据

先停止后台服务：

```bash
systemctl --user disable --now onedrive.service
```

检查是否还有客户端进程：

```bash
pgrep -a onedrive
```

如果没有输出，表示没有运行中的 onedrive 进程。

检查旧账户配置：

```bash
ls -lah ~/.config/onedrive
```

检查旧账户同步目录：

```bash
ls -lah ~/OneDrive
```

确认不再需要这些本地文件后，执行：

```bash
rm -rf ~/.config/onedrive
rm -rf ~/OneDrive
```

检查是否删除成功：

```bash
ls -ld ~/.config/onedrive ~/OneDrive
```

如果显示 `No such file or directory` ，说明本地目录已经删除。

注意：

-   `rm -rf` 不可撤销，执行前必须确认目录路径。
-   删除本地同步目录不会删除 Microsoft 云端文件。
-   删除配置目录会同时删除授权令牌和同步数据库。
-   如果旧账户的本地文件仍有价值，应先备份，不要直接删除。

完成后，重新运行：

```bash
onedrive
```

使用新的 Microsoft 账户授权即可。


## 常见问题与注意事项 {#常见问题与注意事项}


### `--list` 命令不可用 {#list-命令不可用}

本次安装的 2.5.11 版本不支持：

```bash
onedrive --list
```

查看当前版本支持的命令，应使用：

```bash
onedrive --help
```


### `--synchronize` 命令不可用 {#synchronize-命令不可用}

正确命令：

```bash
onedrive --sync
```


### 授权成功但没有执行同步 {#授权成功但没有执行同步}

只运行：

```bash
onedrive
```

主要用于首次授权；授权成功后，它不会自动执行一次完整同步。

需要执行：

```bash
onedrive --sync
```


### 不要随意删除同步数据库 {#不要随意删除同步数据库}

`items.sqlite3` 保存客户端的本地同步状态。不要把它当成普通缓存随意删除。


### 不要随意使用 `--resync` {#不要随意使用-resync}

```bash
onedrive --resync
```

这是重新建立同步状态的操作，不是日常刷新命令。执行前应阅读官方说明，并先进行数据备份。


### 版本更新 {#版本更新}

查看当前版本：

```bash
onedrive --version
```

查看 Fedora 软件仓库中的版本：

```bash
dnf info onedrive
```

更新软件包：

```bash
sudo dnf upgrade onedrive
```

更新后重新检查：

```bash
onedrive --version
onedrive --help
```


## 常用命令速查 {#常用命令速查}

| 操作   | 命令                                             |
|------|------------------------------------------------|
| 查看版本 | `onedrive --version`                             |
| 查看帮助 | `onedrive --help`                                |
| 首次授权 | `onedrive`                                       |
| 查看配置 | `onedrive --display-config`                      |
| 查看云端容量 | `onedrive --display-quota`                       |
| 查看同步状态 | `onedrive --display-sync-status`                 |
| 模拟同步 | `onedrive --sync --dry-run`                      |
| 单次双向同步 | `onedrive --sync`                                |
| 单次只下载 | `onedrive --sync --download-only`                |
| 前台自动监控 | `onedrive --monitor`                             |
| 启动后台服务 | `systemctl --user enable --now onedrive.service` |
| 查看后台服务 | `systemctl --user status onedrive.service`       |
| 查看实时日志 | `journalctl --user -u onedrive.service -f`       |
| 停止后台服务 | `systemctl --user stop onedrive.service`         |
| 重新授权 | `onedrive --reauth`                              |


## 官方参考资料 {#官方参考资料}

-   项目主页：<https://github.com/abraunegg/onedrive>
-   使用说明：<https://github.com/abraunegg/onedrive/blob/master/docs/usage.md>
-   配置选项：<https://github.com/abraunegg/onedrive/blob/master/docs/application-config-options.md>

注意：本文以 Fedora 44 上实际安装的 2.5.11 版本为操作记录。
官方文档的 master 分支可能包含更新版本的选项和行为，使用时应以本机 `onedrive --help` 和对应版本文档为准。

+++
title = "Tex Live 2026 安装"
author = ["zbliang"]
date = 2026-10-07
tags = ["Tex Live"]
categories = ["tech"]
draft = false
+++

## Fedora 上安装 TeX Live：dnf 还是官方安装器？ {#fedora-上安装-tex-live-dnf-还是官方安装器}

在 Fedora 上安装 TeX Live，有两条路可走：

1.  使用 Fedora 的 `dnf` 软件包；
2.  使用 TeX Live 官方安装器 `install-tl` 。

两种方式都能跑通，也各有优点。
它们的区别在于「谁来管理 TeX Live」 。

| 对比项                    | Fedora `dnf`      | TeX Live 官方安装器 |
|------------------------|-------------------|----------------|
| 安装方式                  | `dnf install ...` | `install-tl`    |
| 谁来管理                  | Fedora            | TeX Live 自己   |
| 更新方式                  | 跟随 Fedora       | `tlmgr`         |
| TeX Live 版本             | 取决于 Fedora 版本 | 通常是最新年度版 |
| 宏包更新                  | 由 Fedora 打包后发布 | TeX Live 官方直接更新 |
| 能否安装完整 TeX Live     | 可以              | 可以            |
| 与系统包管理的集成        | 好                | 较弱            |
| 多年保持同一套 TeX Live   | 不方便            | 比较方便        |
| XeLaTeX / LuaLaTeX / 数学宏包 | 都支持            | 都支持          |
| 适用长期科研 / 讲义环境   | 可以              | 更推荐          |

使用 TeX Live 官方安装器还有一个很实际的好处，就是 TeX Live 的管理权不会绑定在 Fedora 上。
所以电脑上可以同时保留多个年度版本，例如：

```bash
/usr/local/texlive/2025/
/usr/local/texlive/2026/
```

然后自行决定默认使用哪一套。


## 总体方案 {#总体方案}

最终我们希望得到这样一套目录：

```bash
/usr/local/texlive/2026/
```

里面是一套完整的 TeX Live 2026。
它的可执行文件目录是：

```bash
/usr/local/texlive/2026/bin/x86_64-linux/
```

把这个目录加入 `PATH` 之后，下面这些命令：

```bash
latex
xelatex
lualatex
tlmgr
```

就都来自这一套官方 TeX Live，而不再是 Fedora 自带的 TeX Live。

之后的更新用：

```bash
tlmgr update --self --all
```

而不是：

```bash
sudo dnf update
```

这样一来，TeX Live 就完全由它自己管理。
官方文档也明确指出，安装完成后，宏包更新与配置由 `tlmgr` 负责。


## 准备 Fedora 上的基础工具 {#准备-fedora-上的基础工具}

打开终端，先安装几个必需的工具：

```bash
sudo dnf install perl wget tar gzip
```

输入 Fedora 用户密码，等待安装完成。

然后检查 Perl 是否可用：

```bash
perl --version
```

如果看到类似输出：

```bash
This is perl 5.xx.x ...
```

就说明 Perl 已经就位。

TeX Live 的 Unix 安装器依赖 Perl 。
官方文档特别说明：在 Unix/Linux 平台上安装，需要系统提供一份标准的 Perl 。


## 下载 TeX Live 2026 安装器 {#下载-tex-live-2026-安装器}

国内用户推荐从清华 TUNA 镜像下载：

```bash
https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet/
```

要下载的文件是：

```bash
install-tl-unx.tar.gz
```

它只有几 MB 大小。


### 解压安装器 {#解压安装器}

```bash
tar -xzf install-tl-unx.tar.gz
```

然后看一下当前目录：

```bash
ls
```

应该能看到类似：

```bash
install-tl-20261002
install-tl-unx.tar.gz
```

这里的日期可能和你看到的不一样，属于正常现象。


### 检查安装器 {#检查安装器}

执行：

```bash
perl install-tl --help
```

如果能显示大段帮助信息，说明安装器工作正常。
也可以看一下版本：

```bash
perl install-tl --version
```


## 开始安装 TeX Live 2026 {#开始安装-tex-live-2026}

```bash
sudo perl install-tl
```

安装器界面出现后，你会看到类似这样的文字：

```bash
======================> TeX Live installation procedure <=====================

...
   <D> directories:
   <S> scheme: full
   <O> options:

Actions:
   <I> start installation to hard disk
   <P> save installation profile to 'texlive.profile'
   <Q> quit
```

此时我们主要检查三件事情。


### Scheme（安装方案） {#scheme-安装方案}

应当是：

```bash
scheme-full
```

或者显示为：

```bash
Selected scheme: full
```

不要改动。


### 安装目录 {#安装目录}

应当类似：

```bash
/usr/local/texlive/2026
```

这也是我们期望的目录。


### 默认纸张 {#默认纸张}

如果你主要用来制作中文数学讲义、Beamer，且国内通常使用 A4，那么保持：

```bash
A4
```

即可。
TeX Live 默认纸张本来就是 A4。


### 启动安装 {#启动安装}

确认以上设置后，在安装器中输入：

```bash
I
```

回车。
接下来就是一段漫长的下载与安装过程。

完整官方安装大约需要 7 GB 以上磁盘空间。


## 安装结束后的检查 {#安装结束后的检查}

安装完成后，检查 TeX Live 是否成功安装。

```bash
ls /usr/local/texlive/2026
```

应该能看到：

```bash
bin
install-tl
readme-html.dir
readme-txt.dir
texmf-dist
tlpkg
```

再看：

```bash
ls /usr/local/texlive/2026/bin
```

应当看到：

```bash
x86_64-linux
```

继续：

```bash
ls /usr/local/texlive/2026/bin/x86_64-linux
```

这里应该至少有：

```bash
latex
xelatex
lualatex
tlmgr
pdflatex
bibtex
biber
...
```


## 配置 PATH {#配置-path}

这是非常关键的一步。

先看看当前使用哪种 shell：

```bash
echo $SHELL
```

Fedora 默认是 Bash，一般会输出：

```bash
/bin/bash
```

如果使用 Zsh，则会输出：

```bash
/bin/zsh
```

下面先按 Bash 来。

打开配置文件：

```bash
nano ~/.bashrc
```

在文件末尾添加一行：

```bash
export PATH=/usr/local/texlive/2026/bin/x86_64-linux:$PATH
```

保存：

```bash
Ctrl + O
```

回车确认，然后：

```bash
Ctrl + X
```

为了让 PATH 立即生效，在终端执行：

```bash
source ~/.bashrc
```

然后检查：

```bash
which latex
```

应当得到：

```bash
/usr/local/texlive/2026/bin/x86_64-linux/latex
```

再验证：

```bash
which xelatex
```

应当得到：

```bash
/usr/local/texlive/2026/bin/x86_64-linux/xelatex
```

以及：

```bash
which tlmgr
```

应当得到：

```bash
/usr/local/texlive/2026/bin/x86_64-linux/tlmgr
```

这三项检查非常重要。

如果这里出现的是：

```bash
/usr/bin/latex
```

而不是：

```bash
/usr/local/texlive/2026/...
```

那就不要继续，先把 `PATH` 的问题解决。


## 检查 TeX Live 版本 {#检查-tex-live-版本}

依次执行：

```bash
latex --version
```

应看到类似：

```bash
pdfTeX ...
TeX Live 2026
```

再执行：

```bash
xelatex --version
```

应看到：

```bash
XeTeX ...
TeX Live 2026
```

最后：

```bash
tlmgr --version
```

应看到：

```bash
tlmgr revision ...
TeX Live (https://tug.org/texlive) version 2026
```

到此可以确认：Fedora 已经完整使用 TeX Live 2026 官方版。


## 第一次更新 `tlmgr` 自身 {#第一次更新-tlmgr-自身}

```bash
tlmgr update --self
```

如果没有问题，再执行：

```bash
tlmgr update --all
```

有一个小提醒：刚装完的 TeX Live 2026 不一定要马上执行 `update --all` 。
因为我们刚刚从 TUNA 镜像下载的，就是当前镜像中最新的网络安装内容。

所以第一次安装成功后，可以先看看有什么可更新的：

```bash
tlmgr update --list
```

如果没有出现类似：

```bash
tlmgr: package repository ...
```

之类的异常提示，那暂时不动也可以。


## 以后如何更新 TeX Live {#以后如何更新-tex-live}

以后基本记住一条命令就够了：

```bash
tlmgr update --self --all
```

如果想先查看有哪些更新：

```bash
tlmgr update --list
```

如果某个宏包缺失，用：

```bash
tlmgr install package-name
```

例如：

```bash
tlmgr install somepackage
```

而 **不要** 用：

```bash
sudo dnf install texlive-somepackage
```

这是因为这台 Fedora 机器上的 TeX Live 已经完全由官方 TeX Live 管理。

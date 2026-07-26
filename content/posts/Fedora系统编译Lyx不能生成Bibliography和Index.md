+++
title = "Fedora系统编译Lyx不能生成Bibliography和Index"
author = ["zbliang"]
date = 2026-07-26
tags = ["Linux", "Lyx"]
categories = ["tech"]
draft = false
+++

有一份数学笔记，是 `.lyx` 格式的文档，在 `macOS` 中可以顺利编译，但在 `Fedora` 系统编译时出错，编译出来的 PDF 文档缺少了 Bibliography 和 Index 这两部分。
点击 `Lyx` 中 `View -> Message Pane` ，把里面的 Messages 复制交给 `Gemini` 分析。
`Gemini` 发现 `Fedora` 缺失了两个系统级 `Perl` 依赖包。

日志中的一处关键错误是：

```text
biber: error while loading shared libraries: libcrypt.so.1: cannot open shared object file: No such file or directory
Systemcall.cpp (305): Systemcall: 'biber "MathNotes"' finished with exit code 127
```

`Gemini` 解释出现这个错误的原因是：较新的 `Fedora` 版本默认使用了更新的 `libxcrypt` 接口 `libcrypt.so.2` ，不再预装旧版的 `libcrypt.so.1` 兼容库。
而 `TeX Live` 附带的 `biber` 可执行文件在 `Linux` 下依赖这个旧版库，导致 `biber` 启动崩溃，参考文献无法处理，所以无法输出 Bibliography（参考文献）。

修复方法是：打开终端，运行以下命令安装兼容库

```text
sudo dnf install libxcrypt-compat
```

日志中第二处关键错误是：

```text
texindy: Can't locate English.pm in @INC (you may need to install the English module)
Systemcall.cpp (305): Systemcall: 'texindy -L english -I xelatex ...' finished with exit code 2
```

报错的原因是： `LyX` 在处理索引时调用了 `xindy` （即 `texindy` ）。
`texindy` 是一个基于 `Perl` 编写的索引处理器。
`Fedora` 系统的 `Perl` 采用了模块化拆分，默认没有预装 `Perl` 的 `English` 模块，导致 `texindy` 脚本直接报 `BEGIN failed` 退出，所以无法生成 Index（索引）。

解决的方法是，在终端运行以下命令安装缺少的 `Perl` 模块（以及 `xindy` 运行时可能需要的 `clisp` 环境）：

```text
sudo dnf install perl-English clisp
```

完成上述两次安装后，回到 `LyX=，点击菜单栏 =Tools -> Reconfigure` ，重启 `LyX` 。
再次编译，发现参考文献 `Biber` 已经完全修复并成功输出了！
（日志显示 `INFO - Output to MathNotes.bbl` ，警告只是一些 `.bib` 文件条目格式的小提示，不影响编译）。
但是 Index 仍然不能输出。

再分析日志，找到关键信息

```text
xindy: Can't locate Env.pm in @INC (you may need to install the Env module)
```

因为 `xindy` 内部调用了多个 `Perl` 模块，上次安装完 `perl-English` 后，它执行到了下一步，又碰到了未预装的 `Env.pm` 模块。

在 `Fedora` 终端中补全安装 `perl-Env` 模块（建议同时把常用的 `Perl` 基础组件一次性补齐，防止以后再弹缺失警告）：

```text
sudo dnf install perl-Env perl-FindBin perl-File-Compare
```

安装后，回到 `LyX=，点击菜单栏 =Tools -> Reconfigure` ，重启 `LyX` 。
这时编译 `.lyx` 文档，成功输出参考文献和索引，问题得到解决。

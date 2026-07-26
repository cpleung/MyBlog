+++
title = "Fedora系统编译Lyx生成多个Okular窗口"
author = ["zbliang"]
date = 2026-07-26
tags = ["Linux", "Lyx"]
categories = ["tech"]
draft = false
+++

在 `Fedora` 系统上编译 `LyX` 遇到一个问题：每编译一次，就新建一个 `Okular` 的窗口。
正常的做法应该是每次编译都是同步原来 `Okular` 窗口的内容，而不应该新建一个窗口。
导致问题的原因是，每次编译导出 PDF 时， `LyX` 都把 `Okular` 当成一个全新的外部程序重新拉起，而不是向已打开的 `Okular` 实例发送重新加载命令。

解决的办法是

-   打开 `LyX` 中的 `Tools -> Preferences` 。
-   在左侧菜单找到 `File Handling -> Viewers` 。
-   在右侧 `Format` 下拉菜单中选择 `PDF (xelatex)` （或者你当前使用的编译器，如 `PDF (pdflatex)` ）。
-   将下面的 `Viewer` 选项修改为：
    ```text
    okular --unique
    ```
    然后保存设置。

这里的参数 `--unique` 告诉系统如果 `Okular` 已经打开了当前文件，只需前置并刷新该窗口，不要新建窗口。

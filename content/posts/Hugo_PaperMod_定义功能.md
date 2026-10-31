+++
title = "Hugo PaperMod 主题的三个自定义功能"
author = ["zbliang"]
date = 2026-10-31
tags = ["Hugo"]
categories = ["tech"]
draft = false
+++

## 修改目标与总体方案 {#修改目标与总体方案}

本次对 Hugo 的 PaperMod 主题进行了三个方面的改进：

1.  ****左侧固定目录（TOC）及当前章节自动高亮**** ：将文章目录放置在正文左侧，并随着页面滚动自动高亮当前阅读章节。
2.  ****长代码块自动折叠**** ：对于超过指定行数的代码块，默认只显示前几行，点击按钮后展开完整代码。
3.  ****图片点击放大（Lightbox）**** ：点击文章中的图片后，在屏幕中央显示高清图片，并提供背景遮罩、关闭按钮和 Esc 关闭功能。

这三个功能分别使用：

-   Hugo 模板覆盖（Template Override）
-   JavaScript
-   CSS

实现。

****最重要的设计原则：不直接修改 PaperMod 主题文件。****

PaperMod 是通过 Git Submodule 安装的，主题目录中的文件属于第三方主题代码。
直接修改这些文件，今后更新主题时可能发生冲突或丢失修改。

因此，本次修改全部放在 Hugo 网站项目自己的目录中，利用 Hugo 的模板查找机制和 PaperMod 提供的扩展接口实现。


## Hugo 项目目录结构 {#hugo-项目目录结构}

假设博客项目位于：

```sh
~/Blog
```

本次修改涉及的目录和文件如下：

```text
MyBlog/
│
├── hugo.yaml
│
├── assets/
│   └── css/
│       └── extended/
│           ├── toc.css
│           ├── code-collapse.css
│           └── image-lightbox.css
│
├── layouts/
│   ├── _partials/
│   │   └── toc.html
│   │
│   └── partials/
│       ├── extend_head.html
│       ├── extend_footer.html
│       └── comments.html
│
├── content/
│   └── ...
│
├── static/
│   └── ...
│
└── themes/
    └── PaperMod/
        └── ...
```

其中：

| 文件或目录                               | 用途                   |
|-------------------------------------|----------------------|
| `hugo.yaml`                              | 配置 PaperMod 的目录显示等参数 |
| `layouts/_partials/toc.html`             | 覆盖 PaperMod 原有的 TOC 模板 |
| `layouts/partials/extend_footer.html`    | 添加三个功能的 JavaScript |
| `assets/css/extended/toc.css`            | 左侧目录布局及高亮样式 |
| `assets/css/extended/code-collapse.css`  | 长代码块折叠样式       |
| `assets/css/extended/image-lightbox.css` | 图片放大样式           |
| `themes/PaperMod/`                       | 原始主题文件，不直接修改 |

****注意：****

Hugo 项目中存在两个容易混淆的目录：

-   `layouts/_partials/` ：本次用于覆盖主题的 `toc.html` 。
-   `layouts/partials/` ：本次用于存放 PaperMod 的扩展脚本 `extend_footer.html` 。

它们不是同一个目录，不能随意互换。

****CSS 文件必须放在 `assets/css/extended/` 中。****

例如，下面的路径是错误的：

```text
assets/css/image-lightbox.css
```

正确路径是：

```text
assets/css/extended/image-lightbox.css
```

PaperMod 会自动加载 `assets/css/extended/` 中的扩展 CSS，并将其纳入主题样式表。


## 第一项改进：左侧固定目录与当前章节自动高亮 {#第一项改进-左侧固定目录与当前章节自动高亮}


### Hugo 配置 {#hugo-配置}

在 `hugo.yaml` 中设置：

```yaml
params:
  UseHugoToc: true
  showtoc: true
  tocopen: true
```

这三个参数的作用：

| 参数         | 作用             |
|------------|----------------|
| `UseHugoToc` | 使用 Hugo 原生目录生成机制 |
| `showtoc`    | 显示文章目录     |
| `tocopen`    | 默认展开目录     |

PaperMod 的参数查找不区分大小写，因此 `showtoc` 与 `ShowToc` 等价。


### 覆盖 PaperMod 原有的 toc.html {#覆盖-papermod-原有的-toc-dot-html}

PaperMod 原本通过 `layouts/_partials/toc.html` 生成文章目录。

为了将目录放在左侧，需要在网站项目中创建同名覆盖文件：

```sh
mkdir -p layouts/_partials
nano layouts/_partials/toc.html
```

添加：

```go-html-template
{{- $headers := findRE "<h[1-6].*?>(.|\n])+?</h[1-6]>" .Content -}}
{{- $has_headers := ge (len $headers) 1 -}}

{{- if $has_headers -}}
<aside class="toc-container wide">
    <details class="toc" {{ if (.Param "TocOpen") }}open{{ end }}>
        <summary accesskey="c" title="(Alt + C)">
            <span class="title">
                {{- i18n "toc" | default "Table of Contents" -}}
            </span>
        </summary>

        <div class="inner">
            {{- if (.Param "UseHugoToc") -}}
                {{- .TableOfContents -}}
            {{- end -}}
        </div>
    </details>
</aside>
{{- end -}}
```

这里的关键是：

```html
<aside class="toc-container wide">
```

它为目录增加了一个外层容器，使 CSS 能够将其定位到文章左侧。


### 左侧目录 CSS {#左侧目录-css}

在 `assets/css/extended/toc.css` 中设置目录样式。

```css
:root {
    --toc-width: 230px;
}

.toc-container {
    margin: 0 2px 40px 2px;
}

.toc {
    border: 1px solid var(--border);
    background: var(--entry);
    border-radius: var(--radius);
    padding: 0.4em;
    margin: 0;
}

/* 目录标题 */
.toc > summary {
    cursor: pointer;
    margin-inline-start: 20px;
    padding: 12px 0;
}

.toc[open] > summary {
    font-weight: 500;
}

/* 目录内容 */
.toc .inner {
    margin: 0 0 0 20px;
    padding: 0 15px 15px 20px;
    font-size: 16px;
    max-height: 70vh;
    overflow-y: auto;
}

.toc ul {
    list-style-type: circle;
}

.toc li ul {
    margin-inline-start: calc(var(--gap) * 0.5);
    list-style-type: none;
}

.toc li {
    list-style: none;
    font-size: 0.95rem;
    padding-bottom: 5px;
}

.toc li a:hover {
    color: var(--secondary);
}

/* 当前章节 */
.toc .active {
    font-size: 110%;
    font-weight: 600;
    color: #614a85;
    text-decoration: underline;
}

/* 滚动条 */
.toc .inner::-webkit-scrollbar-thumb {
    background: var(--border);
    border: 7px solid var(--theme);
    border-radius: var(--radius);
}

/* 大屏幕：目录固定在文章左侧 */
@media screen and (min-width: 1200px) {
    .post-single {
        position: relative;
    }

    .toc-container.wide {
        position: absolute;
        top: 0;
        left: calc(-1 * (var(--toc-width) + var(--gap)));
        width: var(--toc-width);
        height: 100%;
        border-right: 1px solid var(--border);
    }

    .toc-container.wide .toc {
        position: sticky;
        top: var(--gap);
        width: 100%;
        max-height: 83vh;
        overflow-y: auto;
        border: none;
        background: transparent;
        border-radius: 0;
        margin: 0;
    }

    .toc-container.wide .toc .inner {
        margin: 0;
    }
}

/* 小屏幕：目录恢复正常文档流 */
@media screen and (max-width: 1199px) {
    .toc-container.wide {
        position: static;
        width: auto;
        height: auto;
        border: none;
    }

    .toc-container.wide .toc {
        position: static;
        width: auto;
    }
}

/* 当前阅读章节的高亮效果 */
.toc .inner a.active {
    color: var(--primary);
    font-weight: 700;
    text-decoration: none;
    border-left: 3px solid var(--primary);
    padding-left: 8px;
    margin-left: -11px;
}

/* 目录链接过渡 */
.toc .inner a {
    transition: color 0.2s ease;
}

/* 正文标题滚动定位 */
.post-content h1,
.post-content h2,
.post-content h3,
.post-content h4,
.post-content h5,
.post-content h6 {
    scroll-margin-top: 100px;
}
```

其中：

-   `--toc-width` ：控制目录宽度，默认 230px。
-   `1200px` ：控制何时使用左侧布局。
-   `position: sticky` ：使目录在滚动时保持在可视区域内。
-   `position: absolute` ：将目录放置在文章容器左侧。
-   小屏幕下取消左侧定位，避免目录挤压正文。


### Scrollspy：目录随阅读位置自动高亮 {#scrollspy-目录随阅读位置自动高亮}

仅有左侧目录还不够。为了让读者知道当前正在阅读哪一节，需要通过 JavaScript 监听页面滚动。

核心原理：

1.  获取目录中的所有链接。
2.  根据链接的锚点找到正文中对应的标题。
3.  监听浏览器滚动事件。
4.  判断哪个标题已经滚动到页面顶部附近。
5.  给对应目录链接添加 `active` 类。
6.  移除其他链接的 `active` 类。

在 `layouts/partials/extend_footer.html` 中添加：

```javascript
/* =====================================
   1. 目录滚动高亮（Scrollspy）
===================================== */

document.addEventListener("DOMContentLoaded", function () {
    const toc = document.querySelector(".toc-container.wide .toc");

    if (!toc) return;

    // 获取目录中的所有章节链接及其对应的正文标题
    const links = Array.from(
        toc.querySelectorAll('.inner a[href^="#"]')
    );

    const items = links.map(function (link) {
        let id;

        try {
            id = decodeURIComponent(link.hash.substring(1));
        } catch (e) {
            id = link.hash.substring(1);
        }

        return {
            link: link,
            heading: document.getElementById(id)
        };
    }).filter(function (item) {
        return item.heading !== null;
    });

    if (items.length === 0) return;

    let ticking = false;

    // 更新目录的高亮状态
    function updateActiveLink() {
        const offset = 130;
        let activeItem = items[0];

        // 找到当前阅读位置对应的最后一个标题
        for (const item of items) {
            if (item.heading.getBoundingClientRect().top <= offset) {
                activeItem = item;
            } else {
                break;
            }
        }

        // 滚动到文章底部时，高亮最后一个章节
        if (
            window.innerHeight + window.scrollY >=
            document.documentElement.scrollHeight - 2
        ) {
            activeItem = items[items.length - 1];
        }

        // 清除之前的高亮
        items.forEach(function (item) {
            item.link.classList.remove("active");
            item.link.removeAttribute("aria-current");
        });

        // 高亮当前章节
        activeItem.link.classList.add("active");
        activeItem.link.setAttribute("aria-current", "location");
    }

    // 使用 requestAnimationFrame 减少滚动事件的重复计算
    function onScroll() {
        if (!ticking) {
            window.requestAnimationFrame(function () {
                updateActiveLink();
                ticking = false;
            });

            ticking = true;
        }
    }

    window.addEventListener("scroll", onScroll, { passive: true });

    // 页面初次加载时也更新一次
    updateActiveLink();
});
```

其中：

```javascript
const offset = 130;
```

表示以距离浏览器视口顶部约 130px 的位置作为当前章节判断基准。

使用 `requestAnimationFrame` 可以避免在滚动过程中重复执行过多计算。


### 效果与注意事项 {#效果与注意事项}

最终实现：

-   电脑大屏幕上，目录显示在正文左侧。
-   阅读正文时，当前章节自动高亮。
-   向上或向下滚动时，高亮状态实时更新。
-   小屏幕上，目录恢复普通布局。

注意：当前版本实现的是当前目录链接的高亮，并没有额外实现自动展开所有父级目录或自动将高亮条目滚动到目录可视区域。


## 第二项改进：长代码块自动折叠 {#第二项改进-长代码块自动折叠}


### 实现目标 {#实现目标}

当文章中包含大量代码时，全部展开会占用很多页面空间，影响阅读。

因此设置以下规则：

| 参数                 | 默认值 | 作用         |
|--------------------|-----|------------|
| `COLLAPSE_THRESHOLD` | 15  | 超过 15 行自动折叠 |
| `VISIBLE_LINES`      | 8   | 折叠状态下显示前 8 行 |

具体效果：

-   15 行及以下的代码块正常显示。
-   超过 15 行的代码块自动折叠。
-   点击「展开代码」显示全部内容。
-   点击「收起代码」恢复折叠状态。
-   每个代码块独立控制。


### JavaScript {#javascript}

在 `layouts/partials/extend_footer.html` 中追加：

```javascript
/* =====================================
   2. 长代码块自动折叠
===================================== */

document.addEventListener("DOMContentLoaded", function () {
    // 配置参数
    const COLLAPSE_THRESHOLD = 15; // 超过多少行才折叠
    const VISIBLE_LINES = 8;       // 折叠时显示多少行

    // 找到文章中的代码块
    const codeBlocks = document.querySelectorAll(
        ".post-content pre"
    );

    codeBlocks.forEach(function (pre) {
        const code = pre.querySelector("code");

        if (!code) return;

        // 计算代码行数
        const text = code.textContent.replace(/\n$/, "");
        const lineCount = text.split("\n").length;

        // 短代码不折叠
        if (lineCount <= COLLAPSE_THRESHOLD) return;

        // 创建外层容器
        const wrapper = document.createElement("div");
        wrapper.className = "code-collapse";

        // 将 pre 放入容器
        pre.parentNode.insertBefore(wrapper, pre);
        wrapper.appendChild(pre);

        // 设置折叠状态
        wrapper.classList.add("collapsed");
        wrapper.style.setProperty(
            "--visible-lines",
            VISIBLE_LINES
        );

        // 创建展开/收起按钮
        const button = document.createElement("button");
        button.className = "code-collapse-button";
        button.type = "button";
        button.textContent = "展开代码 (" + lineCount + " 行)";
        button.setAttribute("aria-expanded", "false");

        wrapper.appendChild(button);

        // 点击按钮切换状态
        button.addEventListener("click", function () {
            const collapsed = wrapper.classList.toggle("collapsed");

            if (collapsed) {
                button.textContent = "展开代码 (" + lineCount + " 行)";
                button.setAttribute("aria-expanded", "false");
            } else {
                button.textContent = "收起代码";
                button.setAttribute("aria-expanded", "true");
            }
        });
    });
});
```


### CSS {#css}

新建文件：

```sh
nano assets/css/extended/code-collapse.css
```

添加：

```css
/* =====================================
   长代码块自动折叠
===================================== */

/* 代码块外层容器 */
.code-collapse {
    position: relative;
    margin: 1.5em 0;
}

/* 折叠状态 */
.code-collapse.collapsed pre {
    max-height: calc(
        var(--visible-lines) * 1.5em + 1.5em
    );
    overflow: hidden;
}

/* 折叠状态下的渐隐效果 */
.code-collapse.collapsed::after {
    content: "";
    position: absolute;
    left: 0;
    right: 0;
    bottom: 2.8em;
    height: 3em;
    background: linear-gradient(
        to bottom,
        transparent,
        var(--code-block-bg, var(--code-bg))
    );
    pointer-events: none;
}

/* 展开/收起按钮 */
.code-collapse-button {
    display: block;
    width: 100%;
    padding: 0.45em 1em;

    border: 1px solid var(--border);
    border-top: none;
    border-radius: 0 0 var(--radius) var(--radius);

    background: var(--entry);
    color: var(--secondary);

    font-size: 0.85rem;
    cursor: pointer;
    text-align: center;

    transition:
        color 0.2s ease,
        background 0.2s ease;
}

/* 鼠标悬停 */
.code-collapse-button:hover {
    color: var(--primary);
    background: var(--tertiary);
}

/* 键盘焦点 */
.code-collapse-button:focus-visible {
    outline: 2px solid var(--primary);
    outline-offset: 2px;
}
```


### 参数调整 {#参数调整}

如果希望超过 20 行才折叠：

```javascript
const COLLAPSE_THRESHOLD = 20;
```

如果希望折叠时显示 10 行：

```javascript
const VISIBLE_LINES = 10;
```

注意：当前折叠高度使用 CSS 的 `1.5em` 估算每行高度。
如果主题代码块的字体大小或行高发生较大变化，需要相应调整 CSS 中的高度计算。


## 第三项改进：图片点击放大（Lightbox） {#第三项改进-图片点击放大-lightbox}


### 实现目标 {#实现目标}

博客中的图片可能包含大量细节。如果图片只能以正文宽度显示，读者可能无法看清细节。

Lightbox 功能提供以下操作：

-   点击正文图片，打开居中的大图。
-   背景变暗。
-   图片根据屏幕大小自动缩放。
-   点击图片以外的背景区域关闭。
-   点击右上角的关闭按钮关闭。
-   按 `Esc` 键关闭。
-   打开图片时禁止背景页面滚动。
-   鼠标悬停时显示放大镜光标。


### JavaScript {#javascript}

在 `layouts/partials/extend_footer.html` 中追加：

```javascript
/* =====================================
   3. 图片点击放大（Lightbox）
===================================== */

document.addEventListener("DOMContentLoaded", function () {

    // 创建图片放大层
    const lightbox = document.createElement("div");
    lightbox.className = "image-lightbox";
    lightbox.setAttribute("role", "dialog");
    lightbox.setAttribute("aria-modal", "true");
    lightbox.setAttribute("aria-label", "图片预览");
    lightbox.setAttribute("aria-hidden", "true");

    // 图片
    const largeImage = document.createElement("img");
    largeImage.className = "image-lightbox-img";

    // 关闭按钮
    const closeButton = document.createElement("button");
    closeButton.className = "image-lightbox-close";
    closeButton.type = "button";
    closeButton.textContent = "×";
    closeButton.setAttribute("aria-label", "关闭图片预览");

    lightbox.appendChild(largeImage);
    lightbox.appendChild(closeButton);
    document.body.appendChild(lightbox);

    let previousFocus = null;

    // 打开放大图片
    function openLightbox(img) {
        largeImage.src = img.currentSrc || img.src;
        largeImage.alt = img.alt || "";

        lightbox.classList.add("active");
        lightbox.setAttribute("aria-hidden", "false");

        previousFocus = document.activeElement;
        closeButton.focus();

        // 禁止背景页面滚动
        document.body.style.overflow = "hidden";
    }

    // 关闭放大图片
    function closeLightbox() {
        lightbox.classList.remove("active");
        lightbox.setAttribute("aria-hidden", "true");

        largeImage.src = "";

        // 恢复页面滚动
        document.body.style.overflow = "";

        if (previousFocus) {
            previousFocus.focus();
        }
    }

    // 监听文章图片点击
    document.querySelector(".post-content")
        ?.addEventListener("click", function (event) {

            const img = event.target.closest(
                ".post-content img:not(.emoji):not([data-no-lightbox])"
            );

            if (!img) return;

            // 排除代码块中的图片
            if (img.closest("pre, .highlight")) return;

            // 如果图片本身位于链接中，阻止原链接跳转
            const link = img.closest("a");
            if (link) event.preventDefault();

            openLightbox(img);
        });

    // 点击关闭按钮
    closeButton.addEventListener("click", closeLightbox);

    // 点击背景关闭，但点击图片本身不关闭
    lightbox.addEventListener("click", function (event) {
        if (event.target === lightbox) {
            closeLightbox();
        }
    });

    // 按 Esc 关闭
    document.addEventListener("keydown", function (event) {
        if (
            event.key === "Escape" &&
            lightbox.classList.contains("active")
        ) {
            closeLightbox();
        }
    });

});
```

这里使用：

```javascript
img.currentSrc || img.src
```

获取浏览器当前使用的图片资源。

如果以后博客使用缩略图和高清原图分离的方式，可以进一步改进为优先读取图片的 `data-full` 属性。

例如：

```html
<img src="/images/photo-small.jpg"
     data-full="/images/photo-original.jpg"
     alt="图片说明">
```

相应地将 JavaScript 中的：

```javascript
largeImage.src = img.currentSrc || img.src;
```

改为：

```javascript
largeImage.src = img.dataset.full || img.currentSrc || img.src;
```


### CSS {#css}

新建文件：

```sh
nano assets/css/extended/image-lightbox.css
```

添加：

```css
/* =====================================
   文章图片点击放大
===================================== */

/* 文章图片：鼠标悬停时显示放大镜 */
.post-content img:not(.emoji):not([data-no-lightbox]) {
    cursor: zoom-in;
}

/* 图片放大背景 */
.image-lightbox {
    display: none;
    position: fixed;
    inset: 0;
    z-index: 9999;

    justify-content: center;
    align-items: center;

    padding: 40px;

    background: rgba(0, 0, 0, 0.90);

    cursor: zoom-out;
}

/* 激活状态 */
.image-lightbox.active {
    display: flex;
}

/* 放大的图片 */
.image-lightbox-img {
    display: block;

    max-width: 95vw;
    max-height: 92vh;

    width: auto;
    height: auto;

    object-fit: contain;

    border-radius: 4px;

    box-shadow: 0 5px 30px rgba(0, 0, 0, 0.5);

    cursor: default;
}

/* 关闭按钮 */
.image-lightbox-close {
    position: absolute;
    top: 15px;
    right: 25px;

    width: 45px;
    height: 45px;

    border: none;
    border-radius: 50%;

    background: rgba(255, 255, 255, 0.15);
    color: white;

    font-size: 32px;
    line-height: 1;

    cursor: pointer;

    transition: background 0.2s ease;
}

.image-lightbox-close:hover {
    background: rgba(255, 255, 255, 0.30);
}

/* 小屏幕适配 */
@media screen and (max-width: 768px) {
    .image-lightbox {
        padding: 12px;
    }

    .image-lightbox-img {
        max-width: 98vw;
        max-height: 88vh;
    }

    .image-lightbox-close {
        top: 8px;
        right: 8px;
    }
}
```


### 排除不需要放大的图片 {#排除不需要放大的图片}

对于不希望点击放大的图片，可以添加：

```html
<img src="/images/icon.png"
     alt="图标"
     data-no-lightbox>
```

JavaScript 中的选择器会自动排除具有 `data-no-lightbox` 属性的图片。


### 关于 CSS 文件位置的特别提醒 {#关于-css-文件位置的特别提醒}

这次曾经出现过一次问题：

将 `image-lightbox.css` 错误地放在：

```text
assets/css/image-lightbox.css
```

导致 CSS 没有被 PaperMod 自动加载。

正确位置是：

```text
assets/css/extended/image-lightbox.css
```

当 CSS 没有加载时，JavaScript 仍然会创建：

```html
<div class="image-lightbox">
```

但因为没有 `position: fixed=、=display: none` 等样式，图片放大层会像普通 HTML 元素一样出现在文章末尾。

因此，出现这种现象时，应该首先检查 CSS 文件的路径和加载情况，而不是立即修改 JavaScript。


## extend_footer.html 的完整组织方式 {#extend-footer-dot-html-的完整组织方式}

三个功能的 JavaScript 全部放在：

```text
layouts/partials/extend_footer.html
```

文件只需要一组外层 `<script>` 标签。

其结构为：

```html
<script>

/* =====================================
   1. 目录滚动高亮（Scrollspy）
===================================== */

document.addEventListener("DOMContentLoaded", function () {
    // Scrollspy 代码
});


/* =====================================
   2. 长代码块自动折叠
===================================== */

document.addEventListener("DOMContentLoaded", function () {
    // Code Collapse 代码
});


/* =====================================
   3. 图片点击放大（Lightbox）
===================================== */

document.addEventListener("DOMContentLoaded", function () {
    // Lightbox 代码
});

</script>
```

三个 `DOMContentLoaded` 监听器可以独立存在，不需要合并。

需要注意：

-   不要在三个功能之间重复添加或遗漏外层 `<script>` 标签。
-   不要把 JavaScript 放到 `</script>` 标签之外。
-   修改某一个功能时，不要覆盖其他功能的代码。
-   每个功能使用独立的 CSS 文件，便于维护。


## 如何保证更新 PaperMod 后不丢失修改 {#如何保证更新-papermod-后不丢失修改}


### 不直接修改主题目录 {#不直接修改主题目录}

PaperMod 的目录为：

```text
themes/PaperMod/
```

不要直接修改其中的：

```text
themes/PaperMod/layouts/
themes/PaperMod/assets/
```

这些文件属于主题本身。


### 利用 Hugo 模板覆盖机制 {#利用-hugo-模板覆盖机制}

当网站项目和主题中存在相同名称、相同相对路径的模板时，Hugo 会优先使用项目中的模板。

例如，主题原本有：

```text
themes/PaperMod/layouts/_partials/toc.html
```

我们在网站项目中创建：

```text
layouts/_partials/toc.html
```

网站就可以使用自己的目录模板，而不必修改主题文件。

这就是 Template Override（模板覆盖）。


### 利用 PaperMod 扩展接口 {#利用-papermod-扩展接口}

PaperMod 提供了扩展模板接口，例如：

```text
layouts/partials/extend_head.html
layouts/partials/extend_footer.html
```

本次将 JavaScript 放在 `extend_footer.html` 中，就是利用主题提供的扩展接口。

CSS 则放在：

```text
assets/css/extended/
```

由主题自动加载。


### PaperMod 是 Git Submodule {#papermod-是-git-submodule}

当前项目中的 PaperMod 是通过 Git Submodule 安装的。

可以查看：

```sh
git submodule status
```

主题目录由 Git 单独管理。更新主题时，可以执行：

```sh
git submodule update --remote themes/PaperMod
```

更新后再检查：

```sh
hugo --gc
```

由于我们的自定义文件位于项目根目录下的 `layouts/` 和 `assets/` ，而不是 `themes/PaperMod/` 内部，因此正常情况下不会被主题更新覆盖。

不过，主题升级仍可能改变模板调用方式或 CSS 类名，因此升级后应检查三个功能是否仍然正常。


## 日常维护与测试 {#日常维护与测试}


### 检查项目目录 {#检查项目目录}

```sh
cd ~/Blog

find layouts assets/css/extended -type f | sort
```


### 编译检查 {#编译检查}

```sh
hugo --gc
```

如果没有报错，说明 Hugo 模板和资源构建过程通过了基本检查。


### 本地预览 {#本地预览}

```sh
hugo server -D
```

浏览器打开：

```text
http://localhost:1313/
```

修改 CSS 或 JavaScript 后，如果浏览器仍显示旧效果，可以使用：

```text
Ctrl + F5
```

强制刷新页面。


### 三个功能的测试清单 {#三个功能的测试清单}


#### 左侧目录 {#左侧目录}

-   [ ] 大屏幕下目录出现在正文左侧。
-   [ ] 页面向下滚动时目录保持在可视区域。
-   [ ] 当前阅读章节自动高亮。
-   [ ] 向上滚动时高亮状态同步更新。
-   [ ] 点击目录链接可以跳转到对应标题。
-   [ ] 小屏幕下目录不会挤压正文。


#### 长代码块折叠 {#长代码块折叠}

-   [ ] 15 行以内的代码正常显示。
-   [ ] 超过 15 行的代码自动折叠。
-   [ ] 折叠状态下显示前 8 行。
-   [ ] 点击按钮可以展开全部代码。
-   [ ] 再次点击可以收起代码。
-   [ ] 多个代码块可以独立展开和收起。
-   [ ] 语法高亮和代码复制功能正常。


#### 图片点击放大 {#图片点击放大}

-   [ ] 鼠标悬停在文章图片上时显示放大镜光标。
-   [ ] 点击图片后在屏幕中央显示大图。
-   [ ] 背景自动变暗。
-   [ ] 图片不会超出浏览器窗口。
-   [ ] 点击背景可以关闭。
-   [ ] 点击关闭按钮可以关闭。
-   [ ] 按 Esc 可以关闭。
-   [ ] 放大图片时背景页面不会滚动。
-   [ ] 带有 `data-no-lightbox` 的图片不会触发放大。


## 本次修改涉及的文件汇总 {#本次修改涉及的文件汇总}

| 文件                                     | 类型       | 功能           |
|----------------------------------------|----------|--------------|
| `hugo.yaml`                              | 配置       | 开启文章目录   |
| `layouts/_partials/toc.html`             | Hugo 模板覆盖 | 创建左侧目录容器 |
| `layouts/partials/extend_footer.html`    | JavaScript | 目录联动、代码折叠、图片放大 |
| `assets/css/extended/toc.css`            | CSS        | 左侧目录及高亮 |
| `assets/css/extended/code-collapse.css`  | CSS        | 长代码块折叠   |
| `assets/css/extended/image-lightbox.css` | CSS        | 图片放大       |

****维护原则总结：****

1.  Hugo 项目自己的 `layouts/` 用于自定义模板和扩展脚本。
2.  `assets/css/extended/` 用于自定义 CSS。
3.  `themes/PaperMod/` 保持主题原始状态，不直接修改。
4.  使用 Hugo 模板覆盖机制替换需要自定义的主题模板。
5.  使用 PaperMod 提供的扩展接口添加 JavaScript。
6.  升级 PaperMod 后执行 =hugo --gc=，并逐项测试自定义功能。

这样组织项目，可以使自定义功能与主题本身相对独立，便于长期维护和升级。

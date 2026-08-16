+++
title = "Netlify网页增加密码锁"
author = ["zbliang"]
date = 2026-08-16
tags = ["Netlify"]
categories = ["tech"]
draft = false
+++

假设 `GitHub` 仓库的根目录下有一个 `index.html` 文档。
在 `Netlify` 导入该 `GitHub` 仓库，比如名字设置为 `example` ，那么任何人只要在浏览器中打开 `https://example.netlify.app` 后，就都能看到网页 `index.html` 。
能不能给这页面设置一个密码，防止一般人直接访问？
有两种方案。

方案一是采用 `Netlify` 官方提供的内置密码锁功能，但需要付费。
在 `Netlify` 控制台后台进入 `example` 项目，依次进入 `Project configuration` 、 `General` ， `Visitor access` ，就能看到密码设置。
这个方案优点是无需修改任何代码，在服务器端直接拦截，原生弹窗。
但缺点是，使用该功能需要升级到 `Netlify` 付费账号。

方案二是利用 `Netlify Edge Function` 实现免费服务器级密码锁，达到服务器级别的真实拦截（未经授权连 `index.html` 的源代码都拿不到。
方法是现在 `GitHub` 仓库的根目录下新建文件夹结构： `netlify/edge-functions/` 。
然后在目录 `edge-functions` 下新建一个文件 `auth.js` ，粘贴以下代码：

```js
export default async (request, context) => {
  const USERNAME = "admin";    // 账号
  const PASSWORD = "yourpassword123"; // 设置你的密码

  const authHeader = request.headers.get("authorization");

  if (authHeader) {
    const [scheme, encoded] = authHeader.split(" ");
    if (scheme === "Basic") {
      const decoded = atob(encoded);
      const [user, pass] = decoded.split(":");
      if (user === USERNAME && pass === PASSWORD) {
        return await context.next();
      }
    }
  }

  return new Response("Unauthorized", {
    status: 401,
    headers: {
      "WWW-Authenticate": 'Basic realm="Protected Site"',
    },
  });
};

export const config = { path: "/*" };
```

将修改推送到 `GitHub` 后， `Netlify` 会自动识别该函数。
以后任何人访问 `example.netlify.app` 时，浏览器都会弹出一个原生的系统密码框，输入正确才能加载网页内容。

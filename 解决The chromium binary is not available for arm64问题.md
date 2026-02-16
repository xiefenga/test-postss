---
title: 解决The chromium binary is not available for arm64问题
created: 2025-12-16T06:38:42.007Z
updated: 2025-12-16T06:38:42.007Z
slug: 解决The chromium binary is not available for arm64问题
tags: []
description: ""
---

---
title: 解决The chromium binary is not available for arm64问题
created: 2023-6-11 13:36:52
updated: 2023-6-11 13:39:31
---

在基于 ARM 的 Docker 容器（或基于 ARM 的 Linux）中依赖或间接依赖了 `puppeteer` 

在 `npm install ` 的时候会产生 `The chromium binary is not available for arm64` 的错误

一个解决方案：

1. 使用包管理器安装 chromium，例如 `apk add chromium` 
2. 设置环境变量 `PUPPETEER_SKIP_CHROMIUM_DOWNLOAD` 为 `true` 
3. 设置 `PUPPETEER_EXECUTABLE_PATH` 为 `/usr/bin/chromium-browser` 
4. `npm install` 



参考 issue

- [The chromium binary is not available for arm64 #7740 ](https://github.com/puppeteer/puppeteer/issues/7740)

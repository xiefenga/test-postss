---
title: Linux上非root用户使用Docker
created: 2025-12-16T06:38:41.661Z
updated: 2025-12-16T06:38:41.661Z
slug: linuxshang-fei-rootyong-hu-shi-yong-docker
tags: []
description: ""
---

---
title: Linux上非root用户使用Docker
created: 2023-6-12 08:15:12
updated: 2023-6-12 08:15:12
---

Linux 上要让某个非 root 用户能够使用 `Docker`，仅需将该用户添加到 `docker` 用户组

```shell
# 将用户添加到 docker 组
usermod -aG docker <用户名>

# 确认用户已添加到 docker 组
id -nG <用户名>| grep docker

# 重新登录
su - <用户名>
```

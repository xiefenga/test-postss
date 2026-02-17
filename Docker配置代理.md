---
title: Docker配置代理
created: 2025-12-16T06:38:42.255Z
updated: 2025-12-16T06:38:42.255Z
slug: dockerpei-zhi-dai-li
tags: []
description: ""
---

---
title: Docker配置代理
created: 2024-9-23 14:21:20
updated: 2024-9-23 14:22:22
---

在执行 `docker pull` 时，是由守护进程 `dockerd` 来执行。因此，代理需要配在 `dockerd` 的环境中。而这个环境，则是受 `systemd` 所管控，因此实际是 `systemd` 的配置。

```shell
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
```

**proxy.conf**
```shell
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080/"
Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```

**重启**

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

+++
date = '2026-03-03T22:16:52+08:00'
draft = false
title = "Git |Permission denied"
description = "Git |Permission denied"      
summary = "Git |Permission denied"                                                    
categories = ["错误集锦"]                                      
tags = ["Git","ssh","vscode","问题集锦"]                                                  
+++
# vscode上传代码到仓库 |Git |Permission denied

## 错误问题描述

```
Starting ssh-agent on Windows 10 fails: "unable to start ssh-agent service, error :1058"
```

```
OpenSSH_for_Windows_8.1p1, LibreSSL 3.0.2
debug1: Connecting to github.com [] port 22.
debug1: Connection established.
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_rsa type 0
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_rsa-cert type -1
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_dsa type -1
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_dsa-cert type -1
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_ecdsa type -1
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_ecdsa-cert type -1
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_ed25519 type 3
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_ed25519-cert type -1
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_xmss type -1
debug1: identity file C:\\Users\\zhaoyu/.ssh/id_xmss-cert type -1
debug1: Local version string SSH-2.0-OpenSSH_for_Windows_8.1
...
debug1: Authentications that can continue: publickey
debug1: No more authentication methods to try.
git@github.com: Permission denied (publickey).
```

## 解决办法参考

```
PS E:\myblog_hexo\lanan> Get-Service ssh-agent

Status   Name               DisplayName
------   ----               -----------
Stopped  ssh-agent          OpenSSH Authentication Agent

PS E:\myblog_hexo\lanan> Get-Service ssh-agent | Select StartType

StartType
---------
 Disabled


PS E:\myblog_hexo\lanan> Get-Service -Name ssh-agent | Set-Service -StartupType Manual
PS E:\myblog_hexo\lanan> Get-Service ssh-agent | Select StartType

StartType
---------
   Manual
```


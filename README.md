# vul‑study 漏洞复现笔记
> 个人漏洞复现记录，使用 Vulhub 靶场环境，仓库为私有，后续可转为公开。

## 环境准备
### 下载Vulhub靶场
```bash
git clone https://github.com/vulhub/vulhub.git
cd vulhub
- 靶场容器IP：172.18.0.2
- 连接命令：redis-cli -h 172.18.0.2
- 漏洞现象：无需密码直接登录Redis，执行info、keys等命令获取配置与数据

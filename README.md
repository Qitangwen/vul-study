# vul‑study 漏洞复现笔记



## 环境准备

### 安装依赖
```bash
apt update
apt install docker.io docker-compose -y
systemctl start docker
systemctl enable docker
```


### 下载 Vulhub 靶场

```
git clone https://github.com/vulhub/vulhub.git
cd vulhub
```

### 通用操作

```
docker-compose up -d      # 启动靶场
docker-compose down       # 关闭销毁靶场
```

---

## 漏洞列表
1. [Redis未授权访问漏洞](./vulnerabilities/redis-unauthorized/README.md)
2. [Log4j2注入远程代码执行漏洞](./vulnerabilities/log4j2-CVE-2021-44228/README.md)
3. [Tomcat 任意文件上传漏洞](./vulnerabilities/tomcat-cve-2017-12615/README.md)


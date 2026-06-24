---
title: "服务器全方面只读文件审计记录"
tags: ['server', 'audit']
updated: 2026-06-24
---

# 服务器全方面只读文件审计记录

- 审计时间：2026-06-23T13:45:42.806350+00:00
- 审计方式：只读命令、目录统计、配置/服务状态检查、日志摘要检查
- 明确排除：`sub2api` / `sub` 相关业务目录与 `upstream` 相关业务目录，不做代码修改，不做服务修改
- 输出目的：记录可优化点与风险项，供后续人工确认后执行
- 注意：用户要求“逐字逐句审所有文件”，服务器实际文件量包含系统库、node_modules、容器层、缓存、证书归档等，逐字全文人工级审计不现实且可能触及密钥；本记录采用可落地的全盘结构审计 + 高价值配置/日志/权限/体积/服务审计，并避免泄露密钥值。


## 磁盘与目录体积概览

Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/vda1      ext4   40G   18G   23G  45% /
/dev/vdb1      ext4   49G   11G   37G  22% /www

5.4G	/opt
1.5G	/root
1.1G	/var
6.3M	/etc
4.0K	/srv
4.0K	/home

3.4G	/opt/openclaw
697M	/opt/hermes-agent
581M	/opt/shared-agent-memory
228M	/opt/upstream-hub-inspect
154M	/opt/hermes
144M	/opt/src-work
100M	/opt/sub2api
70M	/opt/sub2api.bak.20260622T052801Z
60M	/opt/hermes-daily-summary-web
47M	/opt/hermes-system-monitor
28M	/opt/server-observability-index
8.6M	/opt/upstream-hub-standalone
1.2M	/opt/hermes-dashboard
200K	/opt/server-home
128K	/opt/serverhub
88K	/opt/agent-collab-web
64K	/opt/serverhub-next
60K	/opt/hermes-learning
44K	/opt/obsidian-knowledge-web
40K	/opt/server-file-web
20K	/opt/hermes-docker
12K	/opt/containerd

597M	/root/.local
483M	/root/.hermes
375M	/root/.codex
107M	/root/.openclaw


## 服务与定时器状态

docker-663f67040d5b400f8f79c4ca792df81c3a76de08c47bb7cafaafdb36d436c603.scope transient       -
docker-725cc9f339d1023e1f02f2ecb69a837877aada96b54fef906e45549763c24be6.scope transient       -
docker-9a69b47f645ea85efe4dfabf3c27b3182db6c2ed5f4fdafd71d2d0c3beed461e.scope transient       -
docker-b507619a92678a3ee6e223b3d01efcb9ed60fd930ed6f19c884feba6f585f138.scope transient       -
docker-low-risk-cleanup.service                                               static          -
docker.service                                                                enabled         enabled
hermes-daily-summary-web.service                                              enabled         enabled
hermes-dashboard-web.service                                                  enabled         enabled
hermes-gateway.service                                                        enabled         enabled
hermes-system-monitor.service                                                 enabled         enabled
lvm2-monitor.service                                                          enabled         enabled
mdmonitor-oneshot.service                                                     static          -
mdmonitor.service                                                             static          -
nginx.service                                                                 enabled         enabled
ollama.service                                                                disabled        enabled
openclaw-gateway.service                                                      enabled         enabled
openclaw-hermes-skills-maintenance.service                                    static          -
openclaw-memory-maintenance.service                                           static          -
ua-timer.service                                                              static          -
docker.socket                                                                 enabled         enabled
timers.target                                                                 static          -
apport-autoreport.timer                                                       enabled         enabled
apt-daily-upgrade.timer                                                       enabled         enabled
apt-daily.timer                                                               enabled         enabled
certbot.timer                                                                 enabled         enabled
docker-low-risk-cleanup.timer                                                 enabled         enabled
dpkg-db-backup.timer                                                          enabled         enabled
e2scrub_all.timer                                                             enabled         enabled
fstrim.timer                                                                  enabled         enabled
fwupd-refresh.timer                                                           enabled         enabled
logrotate.timer                                                               enabled         enabled
man-db.timer                                                                  enabled         enabled
mdadm-last-resort@.timer                                                      static          -
mdcheck_continue.timer                                                        enabled         enabled
mdcheck_start.timer                                                           enabled         enabled
mdmonitor-oneshot.timer                                                       enabled         enabled
motd-news.timer                                                               enabled         enabled
openclaw-hermes-skills-maintenance.timer                                      enabled         enabled
openclaw-memory-maintenance.timer                                             enabled         enabled
qwen-organize.timer                                                           disabled        enabled
qwen-warm.timer                                                               disabled        enabled
server-observability-index.timer                                              enabled         enabled
shared-local-task-worker.timer                                                disabled        enabled
shared-memory-vector-index.timer                                              enabled         enabled
snapd.snap-repair.timer                                                       enabled         enabled
systemd-tmpfiles-clean.timer                                                  static          -
ua-license-check.timer                                                        static          -
ua-timer.timer                                                                enabled         enabled
update-notifier-download.timer                                                enabled         enabled
update-notifier-motd.timer                                                    enabled         enabled
xfs_scrub_all.timer                                                           disabled        enabled

Tue 2026-06-23 14:00:00 UTC 14min left    Tue 2026-06-23 12:00:00 UTC 1h 45min ago  openclaw-memory-maintenance.timer        openclaw-memory-maintenance.service
Tue 2026-06-23 14:30:00 UTC 44min left    Tue 2026-06-23 13:30:04 UTC 15min ago     server-observability-index.timer         server-observability-index.service
Tue 2026-06-23 18:34:55 UTC 4h 49min left Tue 2026-06-23 10:33:02 UTC 3h 12min ago  docker-low-risk-cleanup.timer            docker-low-risk-cleanup.service
Tue 2026-06-23 19:02:03 UTC 5h 16min left Tue 2026-06-23 12:15:43 UTC 1h 30min ago  openclaw-hermes-skills-maintenance.timer openclaw-hermes-skills-maintenance.service
Wed 2026-06-24 01:12:09 UTC 11h left      n/a                         n/a           certbot.timer                            certbot.service
Wed 2026-06-24 04:37:49 UTC 14h left      Tue 2026-06-23 04:37:53 UTC 9h ago        shared-memory-vector-index.timer         shared-memory-vector-index.service

  UNIT                                 LOAD   ACTIVE SUB    DESCRIPTION
● fwupd-refresh.service                loaded failed failed Refresh fwupd metadata and update motd
● systemd-networkd-wait-online.service loaded failed failed Wait for Network to be Configured

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
2 loaded units listed.


## 关键服务重启/异常状态

-- hermes-gateway.service
RestartUSec=5s
NRestarts=3182
ExecStart={ path=/opt/hermes-agent/.venv/bin/python ; argv[]=/opt/hermes-agent/.venv/bin/python -m hermes_cli.main gateway run ; ignore_errors=no ; start_time=[Tue 2026-06-23 13:45:46 UTC] ; stop_time=[n/a] ; pid=1184887 ; code=(null) ; status=0/0 }
ActiveState=active
SubState=running
FragmentPath=/etc/systemd/system/hermes-gateway.service
-- openclaw-gateway.service
RestartUSec=5s
NRestarts=0
ExecStart={ path=/root/.local/bin/openclaw ; argv[]=/root/.local/bin/openclaw gateway --port 18789 --verbose ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }
ActiveState=active
SubState=running
FragmentPath=/etc/systemd/system/openclaw-gateway.service
-- hermes-system-monitor.service
RestartUSec=5s
NRestarts=0
ExecStart={ path=/opt/hermes-agent/.venv/bin/python ; argv[]=/opt/hermes-agent/.venv/bin/python main.py ; ignore_errors=no ; start_time=[Tue 2026-06-23 12:32:46 UTC] ; stop_time=[n/a] ; pid=1145751 ; code=(null) ; status=0/0 }
ActiveState=active
SubState=running
FragmentPath=/etc/systemd/system/hermes-system-monitor.service
-- shared-agent-memory.service
RestartUSec=3s
NRestarts=0
ExecStart={ path=/opt/shared-agent-memory/.venv/bin/python ; argv[]=/opt/shared-agent-memory/.venv/bin/python -m uvicorn main:app --host 0.0.0.0 --port 9400 ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }
ActiveState=active
SubState=running
FragmentPath=/etc/systemd/system/shared-agent-memory.service
-- hermes-dashboard-web.service
RestartUSec=5s
NRestarts=0
ExecStart={ path=/opt/hermes-agent/.venv/bin/python ; argv[]=/opt/hermes-agent/.venv/bin/python backend.py ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }
ActiveState=active
SubState=running
FragmentPath=/etc/systemd/system/hermes-dashboard-web.service
-- hermes-daily-summary-web.service
RestartUSec=5s
NRestarts=1
ExecStart={ path=/opt/hermes-daily-summary-web/.venv/bin/python ; argv[]=/opt/hermes-daily-summary-web/.venv/bin/python /opt/hermes-daily-summary-web/backend.py ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }
ActiveState=active
SubState=running
FragmentPath=/etc/systemd/system/hermes-daily-summary-web.service
-- nginx.service
RestartUSec=100ms
NRestarts=0
ExecStart={ path=/usr/sbin/nginx ; argv[]=/usr/sbin/nginx -g daemon on; master_process on; ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }
ActiveState=active
SubState=running
FragmentPath=/lib/systemd/system/nginx.service
-- docker.service
RestartUSec=2s
NRestarts=0
ExecStart={ path=/usr/bin/dockerd ; argv[]=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --data-root=/www/docker ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }
ActiveState=active
SubState=running
FragmentPath=/lib/systemd/system/docker.service


## 网络监听与防火墙

State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess                                                                                                                                                                                                                                                    
LISTEN 0      4096       127.0.0.1:11434      0.0.0.0:*    users:(("ollama",pid=923095,fd=4))                                                                                                                                                                                                                        
LISTEN 0      2048         0.0.0.0:9100       0.0.0.0:*    users:(("python",pid=243253,fd=13))                                                                                                                                                                                                                       
LISTEN 0      4096         0.0.0.0:8080       0.0.0.0:*    users:(("docker-proxy",pid=1012234,fd=8))                                                                                                                                                                                                                 
LISTEN 0      511          0.0.0.0:80         0.0.0.0:*    users:(("nginx",pid=1000307,fd=7),("nginx",pid=1000306,fd=7),("nginx",pid=1000305,fd=7),("nginx",pid=1000304,fd=7),("nginx",pid=1000303,fd=7),("nginx",pid=1000302,fd=7),("nginx",pid=1000301,fd=7),("nginx",pid=1000300,fd=7),("nginx",pid=1000297,fd=7))
LISTEN 0      2048         0.0.0.0:9200       0.0.0.0:*    users:(("python",pid=144100,fd=13))                                                                                                                                                                                                                       
LISTEN 0      5            0.0.0.0:9300       0.0.0.0:*    users:(("python3",pid=131913,fd=3))                                                                                                                                                                                                                       
LISTEN 0      4096   127.0.0.53%lo:53         0.0.0.0:*    users:(("systemd-resolve",pid=52065,fd=14))                                                                                                                                                                                                               
LISTEN 0      128          0.0.0.0:22         0.0.0.0:*    users:(("sshd",pid=125541,fd=3))                                                                                                                                                                                                                          
LISTEN 0      2048         0.0.0.0:9400       0.0.0.0:*    users:(("python",pid=235644,fd=13))                                                                                                                                                                                                                       
LISTEN 0      5            0.0.0.0:7000       0.0.0.0:*    users:(("python3",pid=133254,fd=3))                                                                                                                                                                                                                       
LISTEN 0      511          0.0.0.0:443        0.0.0.0:*    users:(("nginx",pid=1000307,fd=9),("nginx",pid=1000306,fd=9),("nginx",pid=1000305,fd=9),("nginx",pid=1000304,fd=9),("nginx",pid=1000303,fd=9),("nginx",pid=1000302,fd=9),("nginx",pid=1000301,fd=9),("nginx",pid=1000300,fd=9),("nginx",pid=1000297,fd=9))
LISTEN 0      2048         0.0.0.0:9500       0.0.0.0:*    users:(("python",pid=460889,fd=13))                                                                                                                                                                                                                       
LISTEN 0      2048         0.0.0.0:9600       0.0.0.0:*    users:(("python",pid=264117,fd=13))                                                                                                                                                                                                                       
LISTEN 0      2048         0.0.0.0:8000       0.0.0.0:*    users:(("python",pid=262961,fd=13))                                                                                                                                                                                                                       
LISTEN 0      4096         0.0.0.0:8420       0.0.0.0:*    users:(("docker-proxy",pid=1012017,fd=8))                                                                                                                                                                                                                 
LISTEN 0      511        127.0.0.1:18789      0.0.0.0:*    users:(("openclaw",pid=269597,fd=25))                                                                                                                                                                                                                     
LISTEN 0      2048         0.0.0.0:9000       0.0.0.0:*    users:(("python",pid=1145751,fd=17))                                                                                                                                                                                                                      
LISTEN 0      511             [::]:80            [::]:*    users:(("nginx",pid=1000307,fd=8),("nginx",pid=1000306,fd=8),("nginx",pid=1000305,fd=8),("nginx",pid=1000304,fd=8),("nginx",pid=1000303,fd=8),("nginx",pid=1000302,fd=8),("nginx",pid=1000301,fd=8),("nginx",pid=1000300,fd=8),("nginx",pid=1000297,fd=8))
LISTEN 0      128             [::]:22            [::]:*    users:(("sshd",pid=125541,fd=4))                                                                                                                                                                                                                          
LISTEN 0      511             [::]:443           [::]:*    users:(("nginx",pid=1000307,fd=10),("nginx",pid=1000306,fd=10),("nginx",pid=1000305,fd=10),("nginx",pid=1000304,fd=10),("nginx",pid=1000303,fd=10),("nginx",pid=1000302,fd=10),("nginx",pid=1000301,fd=10),("nginx",pid=1000300,fd=10),("nginx",pid=1000297,fd=10))

Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
9000/tcp                   ALLOW IN    Anywhere                  
9100/tcp                   ALLOW IN    Anywhere                  
22/tcp                     ALLOW IN    Anywhere                  
9200/tcp                   ALLOW IN    Anywhere                  
80/tcp                     ALLOW IN    Anywhere                  
9300/tcp                   ALLOW IN    Anywhere                  
7000/tcp                   ALLOW IN    Anywhere                  
9500/tcp                   ALLOW IN    Anywhere                  
9600/tcp                   ALLOW IN    Anywhere                  
443/tcp                    ALLOW IN    Anywhere                  
9000/tcp (v6)              ALLOW IN    Anywhere (v6)             
9100/tcp (v6)              ALLOW IN    Anywhere (v6)             
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
9200/tcp (v6)              ALLOW IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
9300/tcp (v6)              ALLOW IN    Anywhere (v6)             
7000/tcp (v6)              ALLOW IN    Anywhere (v6)             
9500/tcp (v6)              ALLOW IN    Anywhere (v6)             
9600/tcp (v6)              ALLOW IN    Anywhere (v6)             
443/tcp (v6)               ALLOW IN    Anywhere (v6)


## Nginx 与证书

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

total 0
lrwxrwxrwx 1 root root 34 Jun 19 14:22 default -> /etc/nginx/sites-available/default
lrwxrwxrwx 1 root root 38 Jun 22 17:17 zmjjkkk.fun -> /etc/nginx/sites-available/zmjjkkk.fun

/etc/nginx/sites-enabled/default:46:	server_name _;
/etc/nginx/sites-enabled/default:83:#	server_name example.com;
/etc/nginx/sites-enabled/zmjjkkk.fun:13:    server_name monitor.zmjjkkk.fun agent.zmjjkkk.fun notes.zmjjkkk.fun server.zmjjkkk.fun upstream.zmjjkkk.fun subus.zmjjkkk.fun;
/etc/nginx/sites-enabled/zmjjkkk.fun:30:    server_name monitor.zmjjkkk.fun;
/etc/nginx/sites-enabled/zmjjkkk.fun:38:        proxy_pass http://127.0.0.1:9000;
/etc/nginx/sites-enabled/zmjjkkk.fun:50:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-enabled/zmjjkkk.fun:58:        proxy_pass http://127.0.0.1:9100;
/etc/nginx/sites-enabled/zmjjkkk.fun:73:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-enabled/zmjjkkk.fun:81:        proxy_pass http://127.0.0.1:9200;
/etc/nginx/sites-enabled/zmjjkkk.fun:96:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-enabled/zmjjkkk.fun:104:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-enabled/zmjjkkk.fun:116:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-enabled/zmjjkkk.fun:124:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-enabled/zmjjkkk.fun:139:    server_name subus.zmjjkkk.fun;
/etc/nginx/sites-enabled/zmjjkkk.fun:163:        proxy_pass http://127.0.0.1:8080;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:4:    server_name monitor.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:6:        proxy_pass http://127.0.0.1:9000;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:16:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:18:        proxy_pass http://127.0.0.1:9100;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:31:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:33:        proxy_pass http://127.0.0.1:9200;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:46:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:48:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:58:    server_name files.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:60:        proxy_pass http://127.0.0.1:9300;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:71:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.20260622T153713Z:74:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/default:46:	server_name _;
/etc/nginx/sites-available/default:83:#	server_name example.com;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:3:    server_name monitor.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:5:        proxy_pass http://127.0.0.1:9000;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:21:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:23:        proxy_pass http://127.0.0.1:9100;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:42:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:44:        proxy_pass http://127.0.0.1:9200;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:63:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:65:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:82:    server_name files.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:84:        proxy_pass http://127.0.0.1:9300;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:94:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:97:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:118:    server_name subus.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:123:        proxy_pass http://127.0.0.1:8080;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:150:    server_name subus.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:162:    server_name monitor.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:174:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:186:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:199:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:206:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z:209:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:3:    server_name monitor.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:5:        proxy_pass http://127.0.0.1:9000;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:21:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:23:        proxy_pass http://127.0.0.1:9100;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:42:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:44:        proxy_pass http://127.0.0.1:9200;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:63:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:65:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:82:    server_name files.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:84:        proxy_pass http://127.0.0.1:9300;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:94:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:97:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:118:    server_name subus.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:123:        proxy_pass http://127.0.0.1:8080;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:150:    server_name subus.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:162:    server_name monitor.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:174:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:186:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:199:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:206:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:209:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:222:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:226:        proxy_pass http://127.0.0.1:9100;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:238:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:242:        proxy_pass http://127.0.0.1:9200;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:254:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:258:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:267:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208:271:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/zmjjkkk.fun:13:    server_name monitor.zmjjkkk.fun agent.zmjjkkk.fun notes.zmjjkkk.fun server.zmjjkkk.fun upstream.zmjjkkk.fun subus.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun:30:    server_name monitor.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun:38:        proxy_pass http://127.0.0.1:9000;
/etc/nginx/sites-available/zmjjkkk.fun:50:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun:58:        proxy_pass http://127.0.0.1:9100;
/etc/nginx/sites-available/zmjjkkk.fun:73:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun:81:        proxy_pass http://127.0.0.1:9200;
/etc/nginx/sites-available/zmjjkkk.fun:96:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun:104:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/zmjjkkk.fun:116:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun:124:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/zmjjkkk.fun:139:    server_name subus.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun:163:        proxy_pass http://127.0.0.1:8080;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:3:    server_name monitor.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:5:        proxy_pass http://127.0.0.1:9000;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:21:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:23:        proxy_pass http://127.0.0.1:9100;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:42:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:44:        proxy_pass http://127.0.0.1:9200;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:63:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:65:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:82:    server_name files.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:84:        proxy_pass http://127.0.0.1:9300;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:94:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:97:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:118:    server_name subus.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:123:        proxy_pass http://127.0.0.1:8080;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:150:    server_name subus.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:162:    server_name monitor.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:174:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:186:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:199:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:206:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:209:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:224:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:228:        proxy_pass http://127.0.0.1:9100;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:242:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:246:        proxy_pass http://127.0.0.1:9200;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:260:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:264:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:278:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:282:        proxy_pass http://127.0.0.1:8420;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:295:    server_name agent.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:299:        proxy_pass http://127.0.0.1:9100;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:311:    server_name notes.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:315:        proxy_pass http://127.0.0.1:9200;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:327:    server_name server.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:331:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:340:    server_name upstream.zmjjkkk.fun;
/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000:344:        proxy_pass http://127.0.0.1:8420;


- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: agent.zmjjkkk.fun
    Serial Number: 5b58985962f70b39a979cf9d5d6b4885f16
    Key Type: RSA
    Domains: agent.zmjjkkk.fun
    Expiry Date: 2026-09-20 16:17:16+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/agent.zmjjkkk.fun/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/agent.zmjjkkk.fun/privkey.pem
  Certificate Name: monitor.zmjjkkk.fun
    Serial Number: 54a24391e2e1629f76e0b2e3f201d1bdede
    Key Type: RSA
    Domains: monitor.zmjjkkk.fun
    Expiry Date: 2026-09-20 16:17:12+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/monitor.zmjjkkk.fun/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/monitor.zmjjkkk.fun/privkey.pem
  Certificate Name: notes.zmjjkkk.fun
    Serial Number: 56e3e082a5813ec86a73391512761b1e063
    Key Type: RSA
    Domains: notes.zmjjkkk.fun
    Expiry Date: 2026-09-20 16:17:19+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/notes.zmjjkkk.fun/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/notes.zmjjkkk.fun/privkey.pem
  Certificate Name: server.zmjjkkk.fun
    Serial Number: 548fa4e1aef6586d05da4ffc961921b246c
    Key Type: RSA
    Domains: server.zmjjkkk.fun
    Expiry Date: 2026-09-20 16:17:23+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/server.zmjjkkk.fun/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/server.zmjjkkk.fun/privkey.pem
  Certificate Name: subus.zmjjkkk.fun
    Serial Number: 5779a654323e8a251b10474decb7ed7919c
    Key Type: RSA
    Domains: subus.zmjjkkk.fun
    Expiry Date: 2026-09-20 16:17:33+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/subus.zmjjkkk.fun/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/subus.zmjjkkk.fun/privkey.pem
  Certificate Name: upstream.zmjjkkk.fun
    Serial Number: 5c0640e45b183a80ea7b9d3551b6c585a66
    Key Type: RSA
    Domains: upstream.zmjjkkk.fun
    Expiry Date: 2026-09-20 16:17:28+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/upstream.zmjjkkk.fun/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/upstream.zmjjkkk.fun/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -


## 备份/旧文件/可归档文件

backup-like count:
208
backup-like total:
132.65 MB

45338624	/root/server-organization/db-maintenance/20260623T113835Z/backups/hermes-state.db.bak
42434560	/root/server-organization/db-maintenance/20260623T113835Z/backups/system-monitor-metrics.db.bak
23352192	/root/server-organization/db-maintenance/20260623T113835Z/backups/system-monitor-metrics.db-wal.bak
4784128	/root/server-organization/db-maintenance/20260623T113835Z/backups/openclaw-memory.db.bak
4136512	/root/server-organization/db-maintenance/20260623T113835Z/backups/upstream-hub-standalone.db-wal.bak
3139472	/root/server-organization/db-maintenance/20260623T113835Z/backups/hermes-state.db-wal.bak
2240512	/root/server-organization/db-maintenance/20260623T113835Z/backups/openclaw-state.db.bak
990002	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/index-Wjxp3gyC.js
990002	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/index-Wjxp3gyC.js
812308	/opt/hermes-agent/gateway/run.py.bak.openclaw-local-first-20260621060704
749568	/root/server-organization/db-maintenance/20260623T113835Z/backups/shared-agent-memory.db.bak
749568	/opt/shared-agent-memory/data/memory.sqlite.bak.20260623-111818
646872	/root/server-organization/db-maintenance/20260623T113835Z/backups/shared-agent-memory.db-wal.bak
426805	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/index-Dl4jdN7A.css
419499	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/index-Dl4jdN7A.css
300883	/opt/hermes-agent/hermes_cli/config.py.bak-input-cache-20260620034054
253952	/root/server-organization/db-maintenance/20260623T113835Z/backups/upstream-hub-standalone.db.bak
200118	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/markdown-runtime-CUzFp-dW.js
200118	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/markdown-runtime-CUzFp-dW.js
191653	/opt/hermes-agent/gateway/platforms/api_server.py.bak.openclaw-local-first-20260621060704
168128	/root/.openclaw/workspace/hermes-dashboard-index.html.bak.20260622T160818Z
154631	/root/serverhub-migration/backup/20260621-172856/frontend/opt__hermes-dashboard__static/index.html.bak.agent-redesign-20260621165900
154631	/opt/hermes-dashboard/static/index.html.bak.agent-redesign-20260621165900
150122	/root/serverhub-migration/backup/20260621-172856/frontend/opt__hermes-dashboard__static/index.html.bak.agent-ui-20260621152932
150122	/opt/hermes-dashboard/static/index.html.bak.agent-ui-20260621152932
146401	/root/serverhub-migration/backup/20260621-172856/frontend/opt__hermes-dashboard__static/index.html.bak.model-list-fix-20260621151116
146401	/opt/hermes-dashboard/static/index.html.bak.model-list-fix-20260621151116
145481	/root/serverhub-migration/backup/20260621-172856/frontend/opt__hermes-dashboard__static/index.html.bak.model-ui-20260621150035
145481	/opt/hermes-dashboard/static/index.html.bak.model-ui-20260621150035
133120	/root/serverhub-migration/backup/20260621-172856/frontend/opt__hermes-dashboard__static/index.html.bak.20260621050806
133120	/opt/hermes-dashboard/static/index.html.bak.20260621050806
131072	/opt/shared-agent-memory/data/memory.sqlite.bak.20260621044214
114688	/root/server-organization/db-maintenance/20260623T113835Z/backups/hermes-kanban.db.bak
100218	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/agents-BaCrkPrz.js
100031	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/agents-BaCrkPrz.js
96542	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/favicon.ico
96542	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/favicon.ico
89044	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/usage-CJUJgVaq.js
89044	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/usage-CJUJgVaq.js
85814	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/config-runtime-Chws_C6h.js
85814	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/config-runtime-Chws_C6h.js
84958	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/th-DoefQ0YA.js
84958	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/th-DoefQ0YA.js
72245	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/uk-CWxybT97.js
72245	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/uk-CWxybT97.js
68408	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/fa-CrQ2NDPp.js
68408	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/fa-CrQ2NDPp.js
65233	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/ar-t5Ky7OxV.js
65233	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/ar-t5Ky7OxV.js
60504	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/ja-JP-CcWw9Pzm.js
60504	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/ja-JP-CcWw9Pzm.js
57970	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/vi-Qlgs9oO7.js
57970	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/vi-Qlgs9oO7.js
57052	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/cron-BX6W7OhR.js
57052	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/cron-BX6W7OhR.js
54639	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/fr-CppcGv3s.js
54639	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/fr-CppcGv3s.js
54265	/root/.local/lib/node_modules/openclaw/dist/compact-j1-PEvXo.js.bak.local-qwen-routing.20260621053654
54179	/opt/hermes-system-monitor/static/index.html.bak.zh-20260623-121848
53571	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/es-BOjzrrab.js
53571	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/es-BOjzrrab.js
53523	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/ko-BaMqdxR2.js
53523	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/ko-BaMqdxR2.js
53164	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/de-DjvUZ8RA.js
53164	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/de-DjvUZ8RA.js
52814	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/tr-DJEXOZAO.js
52814	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/tr-DJEXOZAO.js
52529	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/pl-CJsiiT1F.js
52529	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/pl-CJsiiT1F.js
52459	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/it-CJo3GflW.js
52459	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/it-CJo3GflW.js
52374	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/string-coerce-DitwMOGn.js
52374	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/string-coerce-DitwMOGn.js
52162	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/pt-BR-Dzoi4PBC.js
52162	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/pt-BR-Dzoi4PBC.js
50683	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/nl-CL2HCQug.js
50683	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/nl-CL2HCQug.js
49697	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/id-Cw-q4WaC.js
49697	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/id-Cw-q4WaC.js
48810	/opt/hermes-system-monitor/static/index.html.bak.extended-20260623T115649Z
48222	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/zh-TW-DQb7_qO9.js
48222	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/zh-TW-DQb7_qO9.js
47233	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/zh-CN-Bu_htjn5.js
47233	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/zh-CN-Bu_htjn5.js
46552	/root/.local/lib/node_modules/openclaw/dist/openai-completions-DCEsPPkI.js.bak.local-first-routing.20260621062447
44716	/opt/hermes-system-monitor/static/index.html.pre-codex-curve-20260623T111759Z
44716	/opt/hermes-system-monitor/static/index.html.bak.20260623111807
43908	/opt/hermes-dashboard/backend.py.bak.model-ui-20260621150035
42310	/opt/hermes-dashboard/backend.py.bak.20260621050806
41678	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/workboard-C6nNtXGt.js
41678	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/workboard-C6nNtXGt.js
39539	/opt/hermes-system-monitor/static/index.html.bak.20260623T110429Z
37714	/opt/hermes-system-monitor/backups/collector.py.pre-backend-opt-20260623T122502Z
36187	/opt/hermes-system-monitor/static/index.html.bak.20260623T092336Z
34701	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/skill-workshop-DYYVlbhT.js
34701	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/skill-workshop-DYYVlbhT.js
34059	/opt/hermes-agent/agent/transports/chat_completions.py.bak-input-cache-20260620034054
32992	/root/server-organization/db-maintenance/20260623T113835Z/backups/openclaw-state.db-wal.bak
32768	/root/server-organization/db-maintenance/20260623T113835Z/backups/upstream-hub-standalone.db-shm.bak
32768	/root/server-organization/db-maintenance/20260623T113835Z/backups/system-monitor-metrics.db-shm.bak
32768	/root/server-organization/db-maintenance/20260623T113835Z/backups/shared-agent-memory.db-shm.bak
32768	/root/server-organization/db-maintenance/20260623T113835Z/backups/openclaw-state.db-shm.bak
32768	/root/server-organization/db-maintenance/20260623T113835Z/backups/openclaw-memory.db-shm.bak
32768	/root/server-organization/db-maintenance/20260623T113835Z/backups/hermes-state.db-shm.bak
28489	/root/.local/lib/node_modules/openclaw/dist/tools-CT_OGlM3.js.bak.20260621040331
26729	/opt/hermes-system-monitor/collector.py.bak.extended-20260623T115649Z
26532	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/sessions-BB4emgwC.js
26532	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/sessions-BB4emgwC.js
26135	/opt/hermes-system-monitor/collector.py.bak.20260623T110637Z
25898	/opt/hermes-system-monitor/collector.py.bak.20260623T110544Z
25155	/root/serverhub-migration/backup/20260622T013049Z/opt/server-home/main.py.bak.20260621015913
25155	/root/serverhub-migration/backup/20260621-172856/frontend/opt__server-home/main.py.bak.20260621015913
25155	/opt/server-home/main.py.bak.20260621015913
24808	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/channels-CwziwylV.js
24808	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/channels-CwziwylV.js
24576	/root/server-organization/db-maintenance/20260623T113835Z/backups/agent-collab.db.bak
23605	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/nodes-C6wprwL9.js
23605	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/nodes-C6wprwL9.js
23121	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/lit-runtime-BImxIzGR.js
23121	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/lit-runtime-BImxIzGR.js
20189	/opt/shared-agent-memory/main.py.bak.20260621044214
19378	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/gateway-runtime-CMyVbEq5.js
19378	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/gateway-runtime-CMyVbEq5.js
17261	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/skills-Bohu9aGR.js
17261	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/skills-Bohu9aGR.js
17233	/opt/hermes-system-monitor/collector.py.bak.20260623T110157Z
16989	/opt/sub2api/backups/upstream-monitor-before-import-20260623T021442Z.sql
16673	/opt/hermes-agent/gateway/delivery.py.bak.openclaw-local-task-20260621084952
13971	/opt/hermes-system-monitor/backups/database.py.pre-backend-opt-20260623T122502Z
13104	/root/.hermes/config.yaml.bak.openclaw-qwen-summary-20260621055104
13080	/root/.hermes/config.yaml.bak-20260620_0737
13049	/root/.hermes/config.yaml.bak-input-cache-20260620034054
13036	/opt/shared-agent-memory/main.py.bak.20260621030638
13026	/opt/agent-collab-web/static/index.html.bak.20260622T164517Z
12477	/root/qwen-cleanup-backups-20260622T050238Z/config.yaml.bak
11730	/opt/sub2api/docker-compose.local.yml.bak.upstream-contract-20260623T043744Z
11730	/opt/sub2api/docker-compose.local.yml.bak.newapi-upstream-contract-20260623T051809Z
11722	/opt/sub2api/docker-compose.local.yml.bak.upstream-monitor-20260622T180623+0000
11720	/opt/sub2api/docker-compose.local.yml.bak.unified-20260622T052930Z
11641	/root/nginx-domain-proxy-backup-20260622T171517+0000/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000
11641	/etc/nginx/sites-available/zmjjkkk.fun.disabled.20260622T171150+0000
11244	/root/.hermes/config.yaml.bak.20260618_094514
10777	/opt/hermes-system-monitor/collector.py.bak.20260623T092159Z
10316	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/index.html
10316	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/index.html
9152	/root/nginx-domain-proxy-backup-20260622T171517+0000/sites-available/zmjjkkk.fun.pre-fix-1782147208
9152	/root/nginx-domain-proxy-backup-20260622T171150+0000/sites-available/zmjjkkk.fun.pre-fix-1782147208
9152	/etc/nginx/sites-available/zmjjkkk.fun.pre-fix-1782147208
7574	/opt/hermes-system-monitor/backups/main.py.pre-backend-opt-20260623T122502Z
6797	/root/nginx-domain-proxy-backup-20260622T171517+0000/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z
6797	/root/nginx-domain-proxy-backup-20260622T171150+0000/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z
6797	/etc/nginx/sites-available/zmjjkkk.fun.bak.fix-ssl.20260622T165033Z
5976	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/activity-MyBG5TwC.js
5976	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/activity-MyBG5TwC.js
5920	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/apple-touch-icon.png
5920	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/apple-touch-icon.png
4624	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/assets/debug-B4oq7IyD.js
4624	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-agents-ui-20260623T113341Z/assets/debug-B4oq7IyD.js
4478	/root/.local/lib/node_modules/openclaw/dist/parallel-search-normalize-DBaknLYY.js
4264	/root/.local/lib/node_modules/openclaw/backups/control-ui.pre-full-redesign-20260623T114601Z/sw.js


## 缓存/临时目录体积

4.2G	/www/data/pnpm/store
563M	/www/data/pnpm/cache
89M	/root/.openclaw/agents
86M	/root/.codex/.tmp
24M	/root/.cache
4.7M	/root/.openclaw/memory
4.3M	/root/.codex/plugins/cache
424K	/root/.local/state


## 权限与危险位检查

world-writable under /opt /root /etc:
/opt/hermes-daily-summary-web/.venv/bin/python3
/opt/hermes-daily-summary-web/.venv/bin/python
/opt/hermes-daily-summary-web/.venv/bin/python3.11
/opt/hermes-daily-summary-web/.venv/lib64
/opt/hermes/.venv/bin/python3
/opt/hermes/.venv/bin/python
/opt/hermes/.venv/bin/python3.11
/opt/hermes/.venv/lib64
/opt/shared-agent-memory/.venv/bin/python3
/opt/shared-agent-memory/.venv/bin/python
/opt/shared-agent-memory/.venv/bin/python3.11
/opt/shared-agent-memory/.venv/lib64
/opt/openclaw/test/helpers/CLAUDE.md
/opt/openclaw/test/CLAUDE.md
/opt/openclaw/extensions/CLAUDE.md
/opt/openclaw/extensions/acpx/CLAUDE.md
/opt/openclaw/docs/CLAUDE.md
/opt/openclaw/docs/reference/templates/CLAUDE.md
/opt/openclaw/src/channels/plugins/contracts/test-helpers/CLAUDE.md
/opt/openclaw/src/channels/CLAUDE.md
/opt/openclaw/src/infra/outbound/CLAUDE.md
/opt/openclaw/src/tui/CLAUDE.md
/opt/openclaw/src/plugins/CLAUDE.md
/opt/openclaw/src/plugin-sdk/CLAUDE.md
/opt/openclaw/src/agents/tools/CLAUDE.md
/opt/openclaw/src/agents/embedded-agent-runner/run/CLAUDE.md
/opt/openclaw/src/agents/CLAUDE.md
/opt/openclaw/src/gateway/server-methods/CLAUDE.md
/opt/openclaw/src/gateway/CLAUDE.md
/opt/openclaw/node_modules/@babel/generator/node_modules/.bin/parser
/opt/openclaw/node_modules/is-expression/node_modules/.bin/acorn
/opt/openclaw/node_modules/node-llama-cpp/node_modules/.bin/node-which
/opt/openclaw/node_modules/node-llama-cpp/node_modules/.bin/nanoid
/opt/openclaw/node_modules/vite/node_modules/.bin/rolldown
/opt/openclaw/node_modules/.bin/parser
/opt/openclaw/node_modules/.bin/node-gyp-build-test
/opt/openclaw/node_modules/.bin/marked
/opt/openclaw/node_modules/.bin/why-is-node-running
/opt/openclaw/node_modules/.bin/node-which
/opt/openclaw/node_modules/.bin/acorn
/opt/openclaw/node_modules/.bin/jsesc
/opt/openclaw/node_modules/.bin/esbuild
/opt/openclaw/node_modules/.bin/vite
/opt/openclaw/node_modules/.bin/resolve
/opt/openclaw/node_modules/.bin/llmock
/opt/openclaw/node_modules/.bin/rolldown
/opt/openclaw/node_modules/.bin/node-edge-tts
/opt/openclaw/node_modules/.bin/openai
/opt/openclaw/node_modules/.bin/oxfmt
/opt/openclaw/node_modules/.bin/aimock
/opt/openclaw/node_modules/.bin/anthropic-ai-sdk
/opt/openclaw/node_modules/.bin/tldts
/opt/openclaw/node_modules/.bin/tsgolint
/opt/openclaw/node_modules/.bin/ciao-bcs
/opt/openclaw/node_modules/.bin/jscpd
/opt/openclaw/node_modules/.bin/tsdown
/opt/openclaw/node_modules/.bin/tsgo
/opt/openclaw/node_modules/.bin/jiti
/opt/openclaw/node_modules/.bin/playwright-core
/opt/openclaw/node_modules/.bin/tsx
/opt/openclaw/node_modules/.bin/tsc
/opt/openclaw/node_modules/.bin/tsserver
/opt/openclaw/node_modules/.bin/nanoid
/opt/openclaw/node_modules/.bin/json5
/opt/openclaw/node_modules/.bin/web-push
/opt/openclaw/node_modules/.bin/node-gyp-build-optional
/opt/openclaw/node_modules/.bin/node-gyp-build
/opt/openclaw/node_modules/.bin/tree-kill
/opt/openclaw/node_modules/.bin/specificity
/opt/openclaw/node_modules/.bin/clawpdf
/opt/openclaw/node_modules/.bin/oxlint
/opt/openclaw/node_modules/.bin/vitest
/opt/openclaw/node_modules/.bin/unrun
/opt/openclaw/node_modules/.bin/qrcode
/opt/openclaw/node_modules/.bin/semver
/opt/openclaw/node_modules/.bin/astring
/opt/openclaw/node_modules/.bin/yaml
/opt/openclaw/node_modules/tsdown/node_modules/.bin/rolldown
/opt/openclaw/node_modules/tree-sitter-bash/node_modules/.bin/node-gyp-build-test
/opt/openclaw/node_modules/tree-sitter-bash/node_modules/.bin/node-gyp-build-optional
/opt/openclaw/node_modules/tree-sitter-bash/node_modules/.bin/node-gyp-build
/opt/openclaw/node_modules/cmake-js/node_modules/.bin/node-which
/opt/openclaw/node_modules/acpx/node_modules/.bin/tsx
/opt/openclaw/node_modules/rolldown-plugin-dts/node_modules/.bin/parser
/opt/openclaw/node_modules/rolldown-plugin-dts/node_modules/.bin/rolldown
/opt/openclaw/node_modules/unrun/node_modules/.bin/rolldown
/opt/openclaw/node_modules/openclaw/node_modules/.bin/openai
/opt/openclaw/node_modules/@npmcli/fs/node_modules/.bin/semver
/opt/openclaw/node_modules/ast-kit/node_modules/.bin/parser
/opt/openclaw/ui/node_modules/.bin/marked
/opt/openclaw/ui/CLAUDE.md
/opt/openclaw/CLAUDE.md
/opt/openclaw/scripts/CLAUDE.md
/opt/hermes-agent/.venv/bin/python3
/opt/hermes-agent/.venv/bin/python
/opt/hermes-agent/.venv/bin/python3.11
/opt/hermes-agent/.venv/lib64
/root/.openclaw/logs
/root/.openclaw/plugin-skills/browser-automation
/root/.openclaw/plugin-skills/wiki-maintainer
/root/.openclaw/plugin-skills/qqbot-remind
/root/.openclaw/plugin-skills/qqbot-media
/root/.openclaw/plugin-skills/qqbot-channel
/root/.openclaw/plugin-skills/obsidian-vault-maintainer
/root/.openclaw/workspace/AGENTS.md
/root/.openclaw/workspace/SHARED_MEMORY.md
/root/.openclaw/workspace/memory/shared-memory.md
/root/.openclaw/workspace/SOUL.md
/root/.openclaw/workspace/IDENTITY.md
/root/.openclaw/workspace/TOOLS.md
/root/.openclaw/workspace/USER.md
/root/.openclaw/npm/projects/openclaw-qqbot-d3553f72f8/node_modules/@openclaw/qqbot/node_modules/.bin/qrcode-terminal
/root/.openclaw/npm/projects/openclaw-qqbot-d3553f72f8/node_modules/@openclaw/qqbot/node_modules/openclaw
/root/serverhub-migration/backup/20260622T013049Z/nginx/etc-nginx/sites-enabled/default
/root/serverhub-migration/backup/20260622T013049Z/nginx/etc-nginx/sites-enabled/zmjjkkk.fun
/root/serverhub-migration/backup/20260622T013049Z/nginx/etc-nginx/modules-enabled/50-mod-stream.conf
/root/serverhub-migration/backup/20260622T013049Z/nginx/etc-nginx/modules-enabled/70-mod-stream-geoip2.conf
/root/serverhub-migration/backup/20260622T013049Z/nginx/etc-nginx/modules-enabled/50-mod-mail.conf
/root/serverhub-migration/backup/20260622T013049Z/nginx/etc-nginx/modules-enabled/50-mod-http-geoip2.conf
/root/serverhub-migration/backup/20260622T013049Z/nginx/etc-nginx/modules-enabled/50-mod-http-image-filter.conf
/root/serverhub-migration/backup/20260622T013049Z/nginx/etc-nginx/modules-enabled/50-mod-http-xslt-filter.conf
/root/serverhub-migration/backup/20260621-172856/nginx/sites-enabled/default
/root/serverhub-migration/backup/20260621-172856/nginx/sites-enabled/zmjjkkk.fun
/root/serverhub-migration/backup/20260621-172831/nginx/sites-enabled/default
/root/serverhub-migration/backup/20260621-172831/nginx/sites-enabled/zmjjkkk.fun
/root/.codex/tmp/arg0/codex-arg0vC6EAL/codex-execve-wrapper
/root/.codex/tmp/arg0/codex-arg0vC6EAL/apply_patch
/root/.codex/tmp/arg0/codex-arg0vC6EAL/applypatch
/root/.codex/tmp/arg0/codex-arg0vC6EAL/codex-linux-sandbox
/root/.codex/packages/standalone/releases/0.142.0-x86_64-unknown-linux-musl/codex
/root/.codex/packages/standalone/current
/root/.hermes/logs
/root/.hermes/lsp/node_modules/.bin/parser
/root/.hermes/lsp/node_modules/.bin/installServerIntoExtension
/root/.hermes/lsp/node_modules/.bin/acorn
/root/.hermes/lsp/node_modules/.bin/vue-language-server
/root/.hermes/lsp/node_modules/.bin/tsc
/root/.hermes/lsp/node_modules/.bin/tsserver
/root/.hermes/lsp/node_modules/.bin/semver
/root/.hermes/lsp/node_modules/.bin/typescript-language-server
/root/.hermes/lsp/bin/vue-language-server
/root/.hermes/lsp/bin/typescript-language-server
/root/.hermes/SHARED_MEMORY.md
/root/.hermes/memories/MEMORY.md
/root/.hermes/memories/SHARED_MEMORY.md
/root/.hermes/memories/USER.md
/root/.hermes/SOUL.md
/root/.hermes/USER.md
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/js-yaml
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/node-which
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/camoufox-js
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/rc
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/playwright-core
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/mime
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/browserslist
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/swagger-jsdoc
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/ua-parser-js
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/prebuild-install
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/semver
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/update-browserslist-db
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/.bin/baseline-browser-mapping
/root/.hermes/node/lib/node_modules/@askjo/camofox-browser/node_modules/swagger-jsdoc/node_modules/.bin/glob
/root/.hermes/node/bin/npx
/root/.hermes/node/bin/agent-browser
/root/.hermes/node/bin/corepack
/root/.hermes/node/bin/camofox-browser
/root/.hermes/node/bin/npm
/root/nginx-domain-proxy-backup-20260622T171517+0000/sites-enabled/default
/root/snap/lxd/current
/root/.agent-browser/browsers
/root/nginx-domain-proxy-backup-20260622T171150+0000/sites-enabled/default
/root/nginx-domain-proxy-backup-20260622T171150+0000/sites-enabled/zmjjkkk.fun
/root/.local/lib/node_modules/openclaw/node_modules/.bin/node-gyp-build-test
/root/.local/lib/node_modules/openclaw/node_modules/.bin/marked
/root/.local/lib/node_modules/openclaw/node_modules/.bin/node-which
/root/.local/lib/node_modules/openclaw/node_modules/.bin/node-edge-tts
/root/.local/lib/node_modules/openclaw/node_modules/.bin/openai
/root/.local/lib/node_modules/openclaw/node_modules/.bin/anthropic-ai-sdk
/root/.local/lib/node_modules/openclaw/node_modules/.bin/ciao-bcs
/root/.local/lib/node_modules/openclaw/node_modules/.bin/jiti
/root/.local/lib/node_modules/openclaw/node_modules/.bin/playwright-core
/root/.local/lib/node_modules/openclaw/node_modules/.bin/tsc
/root/.local/lib/node_modules/openclaw/node_modules/.bin/tsserver
/root/.local/lib/node_modules/openclaw/node_modules/.bin/json5
/root/.local/lib/node_modules/openclaw/node_modules/.bin/web-push
/root/.local/lib/node_modules/openclaw/node_modules/.bin/node-gyp-build-optional
/root/.local/lib/node_modules/openclaw/node_modules/.bin/node-gyp-build
/root/.local/lib/node_modules/openclaw/node_modules/.bin/clawpdf
/root/.local/lib/node_modules/openclaw/node_modules/.bin/qrcode
/root/.local/lib/node_modules/openclaw/node_modules/.bin/yaml
/root/.local/bin/npx
/root/.local/bin/pnx
/root/.local/bin/pn
/root/.local/bin/pnpx
/root/.local/bin/codex
/root/.local/bin/pnpm
/root/.local/bin/npm
/root/.local/bin/openclaw
/root/.local/bin/node
/etc/apparmor/init/network-interface-security/sbin.dhclient
/etc/rmt
/etc/rc1.d/K01nginx
/etc/rc1.d/K01iscsid
/etc/rc1.d/K01open-vm-tools
/etc/rc1.d/K01ufw
/etc/rc1.d/K01lvm2-lvmpolld
/etc/rc1.d/K01multipath-tools
/etc/rc1.d/K01irqbalance
/etc/rc1.d/K01docker
/etc/rc1.d/K01uuidd
/etc/rc1.d/K01open-iscsi
/etc/newt/palette
/etc/modules-load.d/modules.conf
/etc/rc4.d/S01cron
/etc/rc4.d/S01unattended-upgrades
/etc/rc4.d/S01multipath-tools
/etc/rc4.d/S01console-setup.sh
/etc/rc4.d/S01docker
/etc/rc4.d/S01plymouth
/etc/rc4.d/S01uuidd
/etc/rc4.d/S01irqbalance
/etc/rc4.d/S01lvm2-lvmpolld
/etc/rc4.d/S01grub-common
/etc/rc4.d/S01rsync
/etc/rc4.d/S01dbus
/etc/rc4.d/S01apport
/etc/rc4.d/S01open-vm-tools
/etc/rc4.d/S01ssh
/etc/rc4.d/S01nginx
/etc/resolv.conf
/etc/rc2.d/S01cron
/etc/rc2.d/S01unattended-upgrades
/etc/rc2.d/S01multipath-tools
/etc/rc2.d/S01console-setup.sh
/etc/rc2.d/S01docker
/etc/rc2.d/S01plymouth
/etc/rc2.d/S01uuidd
/etc/rc2.d/S01irqbalance
/etc/rc2.d/S01lvm2-lvmpolld
/etc/rc2.d/S01grub-common

setuid/setgid on root filesystem:
4755 root:root /usr/lib/openssh/ssh-keysign
4754 root:messagebus /usr/lib/dbus-1.0/dbus-daemon-launch-helper
2755 root:utmp /usr/lib/x86_64-linux-gnu/utempter/utempter
2755 root:shadow /usr/sbin/pam_extrausers_chkpwd
2755 root:shadow /usr/sbin/unix_chkpwd
2755 root:shadow /usr/bin/expiry
4755 root:root /usr/bin/umount
2755 root:_ssh /usr/bin/ssh-agent
4755 root:root /usr/bin/mount
2755 root:shadow /usr/bin/chage
4755 root:root /usr/bin/passwd
4755 root:root /usr/bin/fusermount3
4755 root:root /usr/bin/chfn
2755 root:crontab /usr/bin/crontab
4755 root:root /usr/bin/su
4755 root:root /usr/bin/pkexec
4755 root:root /usr/bin/newgrp
4755 root:root /usr/bin/chsh
4755 root:root /usr/bin/sudo
4755 root:root /usr/bin/gpasswd
4755 root:root /usr/libexec/polkit-agent-helper-1


## OpenClaw 配置结构（仅键名，不含密钥值）

gateway.mode
gateway.port
plugins.entries.qqbot.enabled
plugins.entries.duckduckgo.enabled
plugins.entries.active-memory.enabled
plugins.entries.browser.enabled
plugins.entries.document-extract.enabled
plugins.entries.file-transfer.enabled
plugins.entries.memory-wiki.enabled
channels.qqbot.enabled
channels.qqbot.appId
channels.qqbot.clientSecret
models.providers.myproxy.api
models.providers.myproxy.baseUrl
models.providers.myproxy.models [list:17]
models.providers.myproxy.apiKey
agents.defaults.model
agents.defaults.memorySearch.enabled
agents.defaults.memorySearch.sources [list:1]
agents.defaults.memorySearch.provider
agents.defaults.memorySearch.fallback
agents.defaults.memorySearch.query.maxResults
agents.defaults.memorySearch.extraPaths [list:0]
agents.defaults.contextInjection
agents.defaults.bootstrapMaxChars
agents.defaults.bootstrapTotalMaxChars
agents.defaults.startupContext.enabled
agents.defaults.startupContext.dailyMemoryDays
agents.defaults.startupContext.maxFileBytes
agents.defaults.startupContext.maxFileChars
agents.defaults.startupContext.maxTotalChars
meta.lastTouchedVersion
meta.lastTouchedAt
tools.web.search.enabled
tools.web.search.provider
tools.web.search.maxResults
tools.web.search.timeoutSeconds
tools.web.search.cacheTtlMinutes

/root/.openclaw/openclaw.json
/root/.openclaw/openclaw.json.backup-before-tools-20260619T235657Z
/root/.openclaw/openclaw.json.backup-before-web-default-20260620T000633Z
/root/.openclaw/openclaw.json.bak
/root/.openclaw/openclaw.json.bak-20260620-021833
/root/.openclaw/openclaw.json.bak.1
/root/.openclaw/openclaw.json.bak.2
/root/.openclaw/openclaw.json.bak.20260621040331
/root/.openclaw/openclaw.json.bak.3
/root/.openclaw/openclaw.json.bak.4
/root/.openclaw/openclaw.json.bak.fix-reply-20260621154729
/root/.openclaw/openclaw.json.bak.local-qwen-routing.20260621053515
/root/.openclaw/openclaw.json.bak.proxy-compat-20260621T162749Z
/root/.openclaw/openclaw.json.bak.switch-proxy-20260621T160442Z
/root/.openclaw/openclaw.json.cache-opt-20260620T045348Z.bak
/root/.openclaw/openclaw.json.clobbered.2026-06-19T15-18-11-092Z
/root/.openclaw/openclaw.json.last-good


## 日志中观测到的异常摘要

hermes-gateway recent:
Jun 23 13:45:31 ser347176424328 python[1184714]: └─────────────────────────────────────────────────────────┘
Jun 23 13:45:31 ser347176424328 python[1184714]: ❌ Gateway already running (PID 977162).
Jun 23 13:45:31 ser347176424328 python[1184714]:    Use 'hermes gateway restart' to replace it,
Jun 23 13:45:31 ser347176424328 python[1184714]:    or 'hermes gateway stop' to kill it first.
Jun 23 13:45:31 ser347176424328 python[1184714]:    Or use 'hermes gateway run --replace' to auto-replace.
Jun 23 13:45:32 ser347176424328 systemd[1]: hermes-gateway.service: Main process exited, code=exited, status=1/FAILURE
Jun 23 13:45:32 ser347176424328 systemd[1]: hermes-gateway.service: Failed with result 'exit-code'.
Jun 23 13:45:32 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 3.573s CPU time.
Jun 23 13:45:37 ser347176424328 systemd[1]: hermes-gateway.service: Scheduled restart job, restart counter is at 3181.
Jun 23 13:45:37 ser347176424328 systemd[1]: Stopped Hermes Agent Gateway - Messaging Platform Integration.
Jun 23 13:45:37 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 3.573s CPU time.
Jun 23 13:45:37 ser347176424328 systemd[1]: Started Hermes Agent Gateway - Messaging Platform Integration.
Jun 23 13:45:40 ser347176424328 python[1184755]: ┌─────────────────────────────────────────────────────────┐
Jun 23 13:45:40 ser347176424328 python[1184755]: │           ⚕ Hermes Gateway Starting...                 │
Jun 23 13:45:40 ser347176424328 python[1184755]: ├─────────────────────────────────────────────────────────┤
Jun 23 13:45:40 ser347176424328 python[1184755]: │  Messaging platforms + cron scheduler                    │
Jun 23 13:45:40 ser347176424328 python[1184755]: │  Press Ctrl+C to stop                                   │
Jun 23 13:45:40 ser347176424328 python[1184755]: └─────────────────────────────────────────────────────────┘
Jun 23 13:45:40 ser347176424328 python[1184755]: ❌ Gateway already running (PID 977162).
Jun 23 13:45:40 ser347176424328 python[1184755]:    Use 'hermes gateway restart' to replace it,
Jun 23 13:45:40 ser347176424328 python[1184755]:    or 'hermes gateway stop' to kill it first.
Jun 23 13:45:40 ser347176424328 python[1184755]:    Or use 'hermes gateway run --replace' to auto-replace.
Jun 23 13:45:41 ser347176424328 systemd[1]: hermes-gateway.service: Main process exited, code=exited, status=1/FAILURE
Jun 23 13:45:41 ser347176424328 systemd[1]: hermes-gateway.service: Failed with result 'exit-code'.
Jun 23 13:45:41 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 3.728s CPU time.
Jun 23 13:45:46 ser347176424328 systemd[1]: hermes-gateway.service: Scheduled restart job, restart counter is at 3182.
Jun 23 13:45:46 ser347176424328 systemd[1]: Stopped Hermes Agent Gateway - Messaging Platform Integration.
Jun 23 13:45:46 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 3.728s CPU time.
Jun 23 13:45:46 ser347176424328 systemd[1]: Started Hermes Agent Gateway - Messaging Platform Integration.
Jun 23 13:45:51 ser347176424328 python[1184887]: ┌─────────────────────────────────────────────────────────┐
Jun 23 13:45:51 ser347176424328 python[1184887]: │           ⚕ Hermes Gateway Starting...                 │
Jun 23 13:45:51 ser347176424328 python[1184887]: ├─────────────────────────────────────────────────────────┤
Jun 23 13:45:51 ser347176424328 python[1184887]: │  Messaging platforms + cron scheduler                    │
Jun 23 13:45:51 ser347176424328 python[1184887]: │  Press Ctrl+C to stop                                   │
Jun 23 13:45:51 ser347176424328 python[1184887]: └─────────────────────────────────────────────────────────┘
Jun 23 13:45:51 ser347176424328 python[1184887]: ❌ Gateway already running (PID 977162).
Jun 23 13:45:51 ser347176424328 python[1184887]:    Use 'hermes gateway restart' to replace it,
Jun 23 13:45:51 ser347176424328 python[1184887]:    or 'hermes gateway stop' to kill it first.
Jun 23 13:45:51 ser347176424328 python[1184887]:    Or use 'hermes gateway run --replace' to auto-replace.
Jun 23 13:45:51 ser347176424328 systemd[1]: hermes-gateway.service: Main process exited, code=exited, status=1/FAILURE
Jun 23 13:45:51 ser347176424328 systemd[1]: hermes-gateway.service: Failed with result 'exit-code'.
Jun 23 13:45:51 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 5.154s CPU time.
Jun 23 13:45:56 ser347176424328 systemd[1]: hermes-gateway.service: Scheduled restart job, restart counter is at 3183.
Jun 23 13:45:56 ser347176424328 systemd[1]: Stopped Hermes Agent Gateway - Messaging Platform Integration.
Jun 23 13:45:56 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 5.154s CPU time.
Jun 23 13:45:56 ser347176424328 systemd[1]: Started Hermes Agent Gateway - Messaging Platform Integration.
Jun 23 13:45:59 ser347176424328 python[1185043]: ┌─────────────────────────────────────────────────────────┐
Jun 23 13:45:59 ser347176424328 python[1185043]: │           ⚕ Hermes Gateway Starting...                 │
Jun 23 13:45:59 ser347176424328 python[1185043]: ├─────────────────────────────────────────────────────────┤
Jun 23 13:45:59 ser347176424328 python[1185043]: │  Messaging platforms + cron scheduler                    │
Jun 23 13:45:59 ser347176424328 python[1185043]: │  Press Ctrl+C to stop                                   │
Jun 23 13:45:59 ser347176424328 python[1185043]: └─────────────────────────────────────────────────────────┘
Jun 23 13:45:59 ser347176424328 python[1185043]: ❌ Gateway already running (PID 977162).
Jun 23 13:45:59 ser347176424328 python[1185043]:    Use 'hermes gateway restart' to replace it,
Jun 23 13:45:59 ser347176424328 python[1185043]:    or 'hermes gateway stop' to kill it first.
Jun 23 13:45:59 ser347176424328 python[1185043]:    Or use 'hermes gateway run --replace' to auto-replace.
Jun 23 13:46:00 ser347176424328 systemd[1]: hermes-gateway.service: Main process exited, code=exited, status=1/FAILURE
Jun 23 13:46:00 ser347176424328 systemd[1]: hermes-gateway.service: Failed with result 'exit-code'.
Jun 23 13:46:00 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 3.390s CPU time.
Jun 23 13:46:05 ser347176424328 systemd[1]: hermes-gateway.service: Scheduled restart job, restart counter is at 3184.
Jun 23 13:46:05 ser347176424328 systemd[1]: Stopped Hermes Agent Gateway - Messaging Platform Integration.
Jun 23 13:46:05 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 3.390s CPU time.
Jun 23 13:46:05 ser347176424328 systemd[1]: Started Hermes Agent Gateway - Messaging Platform Integration.
Jun 23 13:46:10 ser347176424328 python[1185127]: ┌─────────────────────────────────────────────────────────┐
Jun 23 13:46:10 ser347176424328 python[1185127]: │           ⚕ Hermes Gateway Starting...                 │
Jun 23 13:46:10 ser347176424328 python[1185127]: ├─────────────────────────────────────────────────────────┤
Jun 23 13:46:10 ser347176424328 python[1185127]: │  Messaging platforms + cron scheduler                    │
Jun 23 13:46:10 ser347176424328 python[1185127]: │  Press Ctrl+C to stop                                   │
Jun 23 13:46:10 ser347176424328 python[1185127]: └─────────────────────────────────────────────────────────┘
Jun 23 13:46:10 ser347176424328 python[1185127]: ❌ Gateway already running (PID 977162).
Jun 23 13:46:10 ser347176424328 python[1185127]:    Use 'hermes gateway restart' to replace it,
Jun 23 13:46:10 ser347176424328 python[1185127]:    or 'hermes gateway stop' to kill it first.
Jun 23 13:46:10 ser347176424328 python[1185127]:    Or use 'hermes gateway run --replace' to auto-replace.
Jun 23 13:46:11 ser347176424328 systemd[1]: hermes-gateway.service: Main process exited, code=exited, status=1/FAILURE
Jun 23 13:46:11 ser347176424328 systemd[1]: hermes-gateway.service: Failed with result 'exit-code'.
Jun 23 13:46:11 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 5.575s CPU time.
Jun 23 13:46:16 ser347176424328 systemd[1]: hermes-gateway.service: Scheduled restart job, restart counter is at 3185.
Jun 23 13:46:16 ser347176424328 systemd[1]: Stopped Hermes Agent Gateway - Messaging Platform Integration.
Jun 23 13:46:16 ser347176424328 systemd[1]: hermes-gateway.service: Consumed 5.575s CPU time.
Jun 23 13:46:16 ser347176424328 systemd[1]: Started Hermes Agent Gateway - Messaging Platform Integration.

openclaw-gateway recent errors:
Jun 23 13:41:41 ser347176424328 openclaw[269597]: 2026-06-23T13:41:41.979+00:00 [qqbot] gateway closed: 1008 unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:41:41 ser347176424328 openclaw[269597]: 2026-06-23T13:41:41.992+00:00 [ws] closed before connect conn=b16ef173-613b-4de3-98cc-5dba3f9b047f peer=127.0.0.1:42254->127.0.0.1:18789 remote=127.0.0.1 fwd=n/a origin=n/a host=127.0.0.1:18789 ua=n/a code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:41:42 ser347176424328 openclaw[269597]: 2026-06-23T13:41:42.058+00:00 [ws] → close code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token) durationMs=86 cause=unauthorized handshake=failed lastFrameType=req lastFrameMethod=connect lastFrameId=c9dcae1b-e6f6-4d18-a8e9-0717bf5d8de2 endpoint=127.0.0.1:42254->127.0.0.1:18789
Jun 23 13:42:03 ser347176424328 openclaw[269597]: 2026-06-23T13:42:03.573+00:00 [qqbot] [default] [qqbot:api] >>> POST https://api.sgroup.qq.com/v2/users/CA86151BBDA8EFE4C45813BAD0038D2F/messages (timeout: 30000ms)
Jun 23 13:42:09 ser347176424328 openclaw[269597]: 2026-06-23T13:42:09.513+00:00 [provider-transport-fetch] [model-fetch] start provider=myproxy api=openai-completions model=gpt-5.5 method=POST url=https://subus.zmjjkkk.fun/v1/chat/completions timeoutMs=undefined proxy=none policy=custom
Jun 23 13:42:34 ser347176424328 openclaw[269597]: 2026-06-23T13:42:34.143+00:00 [diagnostic] liveness warning: reasons=event_loop_delay interval=54s eventLoopDelayP99Ms=169 eventLoopDelayMaxMs=24629 eventLoopUtilization=0.579 cpuCoreRatio=0.208 active=1 waiting=0 queued=0 recentPhases=sidecars.session-locks:290ms,sidecars.restart-sentinel:283ms,post-attach.update-sentinel:202ms,post-ready.maintenance:51ms,sidecars.model-prewarm:3393ms,post-ready.agent-runtime-plugins:627ms work=[active=agent:main:main(processing/model_call,q=1,age=25s last=model_call:started)]
Jun 23 13:42:34 ser347176424328 openclaw[269597]: 2026-06-23T13:42:34.156+00:00 [qqbot] [default] [qqbot:api] >>> POST https://api.sgroup.qq.com/v2/users/CA86151BBDA8EFE4C45813BAD0038D2F/messages (timeout: 30000ms)
Jun 23 13:42:34 ser347176424328 openclaw[269597]: 2026-06-23T13:42:34.571+00:00 [ws] unauthorized conn=dc55d032-5c12-4a39-9a20-0e4350547458 peer=127.0.0.1:42256->127.0.0.1:18789 remote=127.0.0.1 client=QQ Bot Native Approvals (default) backend v2026.6.8 role=operator scopes=0 auth=token device=no platform=linux instance=n/a host=127.0.0.1:18789 origin=n/a ua=n/a reason=token_missing_config
Jun 23 13:42:34 ser347176424328 openclaw[269597]: 2026-06-23T13:42:34.643+00:00 [qqbot] connect error: unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:42:34 ser347176424328 openclaw[269597]: 2026-06-23T13:42:34.645+00:00 gateway connect failed: GatewayClientRequestError: unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:42:34 ser347176424328 openclaw[269597]: 2026-06-23T13:42:34.916+00:00 [qqbot] gateway closed: 1008 unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:42:34 ser347176424328 openclaw[269597]: 2026-06-23T13:42:34.944+00:00 [ws] closed before connect conn=dc55d032-5c12-4a39-9a20-0e4350547458 peer=127.0.0.1:42256->127.0.0.1:18789 remote=127.0.0.1 fwd=n/a origin=n/a host=127.0.0.1:18789 ua=n/a code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:42:34 ser347176424328 openclaw[269597]: 2026-06-23T13:42:34.980+00:00 [ws] → close code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token) durationMs=451 cause=unauthorized handshake=failed lastFrameType=req lastFrameMethod=connect lastFrameId=1ecfa7d5-93e5-4852-920b-e23f1bfc9862 endpoint=127.0.0.1:42256->127.0.0.1:18789
Jun 23 13:42:36 ser347176424328 openclaw[269597]: 2026-06-23T13:42:36.773+00:00 [qqbot] [default] [qqbot:api] >>> POST https://api.sgroup.qq.com/v2/users/CA86151BBDA8EFE4C45813BAD0038D2F/messages (timeout: 30000ms)
Jun 23 13:42:48 ser347176424328 openclaw[269597]: 2026-06-23T13:42:48.944+00:00 [provider-transport-fetch] [model-fetch] start provider=myproxy api=openai-completions model=gpt-5.5 method=POST url=https://subus.zmjjkkk.fun/v1/chat/completions timeoutMs=undefined proxy=none policy=custom
Jun 23 13:42:53 ser347176424328 openclaw[269597]: 2026-06-23T13:42:53.696+00:00 [provider-transport-fetch] [model-fetch] start provider=myproxy api=openai-completions model=gpt-5.5 method=POST url=https://subus.zmjjkkk.fun/v1/chat/completions timeoutMs=undefined proxy=none policy=custom
Jun 23 13:45:16 ser347176424328 openclaw[269597]: 2026-06-23T13:45:16.124+00:00 [diagnostic] liveness warning: reasons=event_loop_delay,event_loop_utilization interval=162s eventLoopDelayP99Ms=305.7 eventLoopDelayMaxMs=142539.2 eventLoopUtilization=0.953 cpuCoreRatio=0.076 active=1 waiting=0 queued=0 recentPhases=sidecars.session-locks:290ms,sidecars.restart-sentinel:283ms,post-attach.update-sentinel:202ms,post-ready.maintenance:51ms,sidecars.model-prewarm:3393ms,post-ready.agent-runtime-plugins:627ms work=[active=agent:main:qqbot:default:direct:ca86151bbda8efe4c45813bad0038d2f(processing/model_call,q=1,age=159s last=model_call:started)]
Jun 23 13:45:16 ser347176424328 openclaw[269597]: 2026-06-23T13:45:16.563+00:00 [ws] unauthorized conn=856d3cf9-ef12-40e1-a386-0c343d5c5e9f peer=127.0.0.1:42258->127.0.0.1:18789 remote=127.0.0.1 client=QQ Bot Native Approvals (default) backend v2026.6.8 role=operator scopes=0 auth=token device=no platform=linux instance=n/a host=127.0.0.1:18789 origin=n/a ua=n/a reason=token_missing_config
Jun 23 13:45:16 ser347176424328 openclaw[269597]: 2026-06-23T13:45:16.603+00:00 [qqbot] connect error: unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:45:16 ser347176424328 openclaw[269597]: 2026-06-23T13:45:16.604+00:00 gateway connect failed: GatewayClientRequestError: unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:45:16 ser347176424328 openclaw[269597]: 2026-06-23T13:45:16.770+00:00 [qqbot] gateway closed: 1008 unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:45:16 ser347176424328 openclaw[269597]: 2026-06-23T13:45:16.804+00:00 [ws] closed before connect conn=856d3cf9-ef12-40e1-a386-0c343d5c5e9f peer=127.0.0.1:42258->127.0.0.1:18789 remote=127.0.0.1 fwd=n/a origin=n/a host=127.0.0.1:18789 ua=n/a code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:45:16 ser347176424328 openclaw[269597]: 2026-06-23T13:45:16.815+00:00 [ws] → close code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token) durationMs=328 cause=unauthorized handshake=failed lastFrameType=req lastFrameMethod=connect lastFrameId=71bebf03-c676-4119-9fc5-6b29fa51ba1d endpoint=127.0.0.1:42258->127.0.0.1:18789
Jun 23 13:45:17 ser347176424328 openclaw[269597]: 2026-06-23T13:45:17.543+00:00 [qqbot] [default] [qqbot:api] >>> GET https://api.sgroup.qq.com/gateway (timeout: 30000ms)
Jun 23 13:45:42 ser347176424328 openclaw[269597]: 2026-06-23T13:45:42.415+00:00 [qqbot] [default] [qqbot:api] >>> POST https://api.sgroup.qq.com/v2/users/CA86151BBDA8EFE4C45813BAD0038D2F/messages (timeout: 30000ms)
Jun 23 13:45:46 ser347176424328 openclaw[269597]: 2026-06-23T13:45:46.833+00:00 [ws] unauthorized conn=84e6d30c-3bd6-4161-b066-f789b9b225fe peer=127.0.0.1:42260->127.0.0.1:18789 remote=127.0.0.1 client=QQ Bot Native Approvals (default) backend v2026.6.8 role=operator scopes=0 auth=token device=no platform=linux instance=n/a host=127.0.0.1:18789 origin=n/a ua=n/a reason=token_missing_config
Jun 23 13:45:46 ser347176424328 openclaw[269597]: 2026-06-23T13:45:46.840+00:00 [qqbot] connect error: unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:45:46 ser347176424328 openclaw[269597]: 2026-06-23T13:45:46.841+00:00 gateway connect failed: GatewayClientRequestError: unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:45:46 ser347176424328 openclaw[269597]: 2026-06-23T13:45:46.852+00:00 [qqbot] gateway closed: 1008 unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:45:46 ser347176424328 openclaw[269597]: 2026-06-23T13:45:46.914+00:00 [ws] closed before connect conn=84e6d30c-3bd6-4161-b066-f789b9b225fe peer=127.0.0.1:42260->127.0.0.1:18789 remote=127.0.0.1 fwd=n/a origin=n/a host=127.0.0.1:18789 ua=n/a code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:45:46 ser347176424328 openclaw[269597]: 2026-06-23T13:45:46.920+00:00 [ws] → close code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token) durationMs=70 cause=unauthorized handshake=failed lastFrameType=req lastFrameMethod=connect lastFrameId=b6b53619-54f1-488e-92b5-073fd4f7f1e1 endpoint=127.0.0.1:42260->127.0.0.1:18789
Jun 23 13:45:53 ser347176424328 openclaw[269597]: 2026-06-23T13:45:53.304+00:00 [provider-transport-fetch] [model-fetch] start provider=myproxy api=openai-completions model=gpt-5.5 method=POST url=https://subus.zmjjkkk.fun/v1/chat/completions timeoutMs=undefined proxy=none policy=custom
Jun 23 13:46:16 ser347176424328 openclaw[269597]: 2026-06-23T13:46:16.894+00:00 [ws] unauthorized conn=a6c7ae9d-b218-4ce1-866e-97579106f5b8 peer=127.0.0.1:42264->127.0.0.1:18789 remote=127.0.0.1 client=QQ Bot Native Approvals (default) backend v2026.6.8 role=operator scopes=0 auth=token device=no platform=linux instance=n/a host=127.0.0.1:18789 origin=n/a ua=n/a reason=token_missing_config
Jun 23 13:46:16 ser347176424328 openclaw[269597]: 2026-06-23T13:46:16.900+00:00 [qqbot] connect error: unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:46:16 ser347176424328 openclaw[269597]: 2026-06-23T13:46:16.901+00:00 gateway connect failed: GatewayClientRequestError: unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:46:16 ser347176424328 openclaw[269597]: 2026-06-23T13:46:16.923+00:00 [qqbot] gateway closed: 1008 unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:46:16 ser347176424328 openclaw[269597]: 2026-06-23T13:46:16.934+00:00 [ws] closed before connect conn=a6c7ae9d-b218-4ce1-866e-97579106f5b8 peer=127.0.0.1:42264->127.0.0.1:18789 remote=127.0.0.1 fwd=n/a origin=n/a host=127.0.0.1:18789 ua=n/a code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token)
Jun 23 13:46:16 ser347176424328 openclaw[269597]: 2026-06-23T13:46:16.943+00:00 [ws] → close code=1008 reason=unauthorized: gateway token not configured on gateway (set gateway.auth.token) durationMs=56 cause=unauthorized handshake=failed lastFrameType=req lastFrameMethod=connect lastFrameId=bbcf379a-bb96-4385-af6e-40276b1df7cd endpoint=127.0.0.1:42264->127.0.0.1:18789

docker recent errors:
Jun 23 02:50:42 ser347176424328 dockerd[270255]: time="2026-06-23T02:50:42.384481166Z" level=warning msg="healthcheck failed" actualDuration="694.554µs" error="Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\"" timeout=15s
Jun 23 02:50:47 ser347176424328 dockerd[270255]: time="2026-06-23T02:50:47.382603259Z" level=error msg="healthcheck failed fatally" error="session healthcheck failed fatally: Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\""
Jun 23 03:08:26 ser347176424328 dockerd[270255]: time="2026-06-23T03:08:26.122587774Z" level=warning msg="healthcheck failed" actualDuration=1.126255ms error="Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\"" timeout=15s
Jun 23 03:08:31 ser347176424328 dockerd[270255]: time="2026-06-23T03:08:31.158258345Z" level=error msg="healthcheck failed fatally" error="session healthcheck failed fatally: Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\""
Jun 23 03:18:20 ser347176424328 dockerd[270255]: time="2026-06-23T03:18:20.252218106Z" level=warning msg="failed to read oom_kill event" error="open /sys/fs/cgroup/system.slice:docker:dtqoxqak6n98f8q8r2y27o3ov/memory.events: no such file or directory" span="[backend-builder 8/8] RUN VERSION_VALUE=\"\" &&     if [ -z \"${VERSION_VALUE}\" ]; then VERSION_VALUE=\"$(tr -d '\\r\\n' < ./cmd/server/VERSION)\"; fi &&     DATE_VALUE=\"${DATE:-$(date -u +%Y-%m-%dT%H:%M:%SZ)}\" &&     CGO_ENABLED=0 GOOS=linux go build     -tags embed     -ldflags=\"-s -w -X main.Version=${VERSION_VALUE} -X main.Commit=docker -X main.Date=${DATE_VALUE} -X main.BuildType=release\"     -trimpath     -o /app/sub2api     ./cmd/server" spanID=ffa56bcdb0ed6af1 traceID=cc81a9285edc55c76277194224ae392a
Jun 23 03:18:20 ser347176424328 dockerd[270255]: time="2026-06-23T03:18:20.969500267Z" level=error msg=/moby.buildkit.v1.Control/Solve error="rpc error: code = Canceled desc = context canceled" spanID=eafbb6be19cc8cae traceID=cc81a9285edc55c76277194224ae392a
Jun 23 03:18:21 ser347176424328 dockerd[270255]: time="2026-06-23T03:18:21.234789797Z" level=warning msg="healthcheck failed" actualDuration="715.674µs" error="Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\"" timeout=15s
Jun 23 03:18:26 ser347176424328 dockerd[270255]: time="2026-06-23T03:18:26.239399505Z" level=error msg="healthcheck failed fatally" error="session healthcheck failed fatally: Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\""
Jun 23 03:18:39 ser347176424328 dockerd[270255]: time="2026-06-23T03:18:39.209827499Z" level=warning msg="healthcheck failed" actualDuration=5.789558ms error="Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\"" timeout=15s
Jun 23 03:18:44 ser347176424328 dockerd[270255]: time="2026-06-23T03:18:44.206182692Z" level=error msg="healthcheck failed fatally" error="session healthcheck failed fatally: Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\""
Jun 23 03:23:04 ser347176424328 dockerd[270255]: time="2026-06-23T03:23:04.267382018Z" level=warning msg="healthcheck failed" actualDuration=1.112509ms error="Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\"" timeout=15s
Jun 23 03:23:09 ser347176424328 dockerd[270255]: time="2026-06-23T03:23:09.266050432Z" level=error msg="healthcheck failed fatally" error="session healthcheck failed fatally: Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\""
Jun 23 04:28:36 ser347176424328 dockerd[270255]: time="2026-06-23T04:28:36.558059894Z" level=warning msg="healthcheck failed" actualDuration="798.395µs" error="Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\"" timeout=15s
Jun 23 04:28:41 ser347176424328 dockerd[270255]: time="2026-06-23T04:28:41.544257321Z" level=error msg="healthcheck failed fatally" error="session healthcheck failed fatally: Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\""
Jun 23 04:37:11 ser347176424328 dockerd[270255]: time="2026-06-23T04:37:11.689194776Z" level=warning msg="healthcheck failed" actualDuration="959.756µs" error="Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\"" timeout=15s
Jun 23 04:37:16 ser347176424328 dockerd[270255]: time="2026-06-23T04:37:16.688281250Z" level=error msg="healthcheck failed fatally" error="session healthcheck failed fatally: Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\""
Jun 23 05:13:40 ser347176424328 dockerd[270255]: 2026/06/23 05:13:40 http2: server: error reading preface from client @: read unix /run/docker.sock->@: read: connection reset by peer
Jun 23 05:13:45 ser347176424328 dockerd[270255]: time="2026-06-23T05:13:45.194669965Z" level=warning msg="healthcheck failed" actualDuration="934.645µs" error="Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\"" timeout=15s
Jun 23 05:13:50 ser347176424328 dockerd[270255]: time="2026-06-23T05:13:50.194090696Z" level=error msg="healthcheck failed fatally" error="session healthcheck failed fatally: Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\""
Jun 23 05:18:10 ser347176424328 dockerd[270255]: time="2026-06-23T05:18:10.338447926Z" level=warning msg="healthcheck failed" actualDuration="481.758µs" error="Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\"" timeout=15s
Jun 23 05:18:15 ser347176424328 dockerd[270255]: time="2026-06-23T05:18:15.344569383Z" level=error msg="healthcheck failed fatally" error="session healthcheck failed fatally: Unavailable: connection error: desc = \"transport: Error while dialing: only one connection allowed\""
Jun 23 07:13:45 ser347176424328 dockerd[270255]: time="2026-06-23T07:13:45.948424974Z" level=info msg="Container failed to exit within 10s of signal 15 - using the force" container=9a69b47f645ea85efe4dfabf3c27b3182db6c2ed5f4fdafd71d2d0c3beed461e
Jun 23 07:13:46 ser347176424328 dockerd[270255]: time="2026-06-23T07:13:46.610609228Z" level=info msg="stopping event stream following graceful shutdown" error="<nil>" module=libcontainerd namespace=moby
Jun 23 07:13:46 ser347176424328 dockerd[270255]: time="2026-06-23T07:13:46.627131238Z" level=info msg="stopping event stream following graceful shutdown" error="context canceled" module=libcontainerd namespace=plugins.moby
Jun 23 07:30:13 ser347176424328 dockerd[1011546]: time="2026-06-23T07:30:13.678568306Z" level=info msg="Creating a containerd client" address=/run/containerd/containerd.sock timeout=1m0s
Jun 23 07:30:13 ser347176424328 dockerd[1011546]: time="2026-06-23T07:30:13.992489886Z" level=info msg="Deleting nftables IPv4 rules" error="running nft: /dev/stdin:1:17-30: Error: Could not process rule: No such file or directory\ndelete table ip docker-bridges\n                ^^^^^^^^^^^^^^\n exit status 1"
Jun 23 07:30:14 ser347176424328 dockerd[1011546]: time="2026-06-23T07:30:14.026823255Z" level=info msg="Deleting nftables IPv6 rules" error="running nft: /dev/stdin:1:18-31: Error: Could not process rule: No such file or directory\ndelete table ip6 docker-bridges\n                 ^^^^^^^^^^^^^^\n exit status 1"
Jun 23 10:30:42 ser347176424328 dockerd[1011546]: time="2026-06-23T10:30:42.368086157Z" level=error msg="[resolver] failed to query external DNS server" client-addr="udp:127.0.0.1:51660" dns-server="udp:127.0.0.53:53" error="read udp 127.0.0.1:51660->127.0.0.53:53: i/o timeout" question=";api.capsolver.com.\tIN\t A"
Jun 23 10:30:42 ser347176424328 dockerd[1011546]: time="2026-06-23T10:30:42.368701257Z" level=error msg="[resolver] failed to query external DNS server" client-addr="udp:127.0.0.1:59101" dns-server="udp:127.0.0.53:53" error="read udp 127.0.0.1:59101->127.0.0.53:53: i/o timeout" question=";api.capsolver.com.\tIN\t AAAA"


## 本地端口健康探测

http://127.0.0.1:9000/api/health -> 200
http://127.0.0.1:9100/health -> 404
http://127.0.0.1:9200/health -> 404
http://127.0.0.1:8000/health -> 200
http://127.0.0.1:9400/health -> 200
http://127.0.0.1:9500/health -> 404
http://127.0.0.1:9600/health -> 200


## 审计发现与优化建议

### P0 / 高优先级

1. **`hermes-gateway.service` 重启次数异常高**
   - 证据：`NRestarts=3162`，日志显示 `Gateway already running (PID 977162)`，systemd 每 5 秒拉起失败/再拉起。
   - 影响：持续制造日志、CPU 抖动、服务状态误报；也可能与当前 OpenClaw/QQBot Gateway auth 问题相互干扰。
   - 建议：先人工确认旧 PID 977162 的来源与是否仍需要，再统一 systemd unit 的启动方式；不要直接 kill，避免误断消息入口。

2. **OpenClaw gateway auth token 未配置导致内部 ws 授权错误反复出现**
   - 证据：`unauthorized: gateway token not configured on gateway (set gateway.auth.token)`。
   - 影响：QQBot native approvals / Gateway cron 等能力异常；此前 Gateway cron 已因 auth 缺失不可用。
   - 建议：按 OpenClaw schema 设置 `gateway.auth.token` 与对应客户端配置；配置前先查 schema，变更后用 gateway 工具热加载/重启。

3. **公网暴露端口偏多**
   - 证据：UFW 放行 `7000/9000/9100/9200/9300/9500/9600` 等，监听为 `0.0.0.0`。
   - 影响：监控/内部 Web/API 若无强认证，暴露面偏大。
   - 建议：优先改为只经 Nginx HTTPS 反代访问，后端服务监听 `127.0.0.1` 或 UFW 限源。此项需逐项确认，不能批量关闭。

### P1 / 中优先级

4. **备份/旧版本文件较多，可归档而不是长期散落在生产目录**
   - 证据：backup-like 文件总计约 `133.06 MB`，包括 `/opt/hermes-system-monitor/*.bak*`、Nginx 多个 `.bak/.disabled/pre-fix`、OpenClaw dist 备份等。
   - 影响：搜索/审计噪声大，误编辑旧文件风险增加。
   - 建议：统一移动到 `/root/server-organization/audit-archive/YYYYMMDD/` 或打包压缩；保留最近可回滚点。

5. **pnpm store/cache 占用较大**
   - 证据：`/www/data/pnpm/store` 约 `4.2G`，`/www/data/pnpm/cache` 约 `563M`。
   - 影响：长期增长会占用 `/www`，虽当前 `/www` 使用率只有约 22%。
   - 建议：后续可用 pnpm prune/store prune 策略，但要确认哪些项目依赖该 store，不建议直接删除。

6. **`/opt/openclaw/node_modules` 占用 3.1G**
   - 证据：`/opt/openclaw` 约 3.4G，其中 `node_modules` 约 3.1G。
   - 影响：可接受但偏大；升级/备份成本高。
   - 建议：如 `/opt/openclaw` 是源码树而运行实际使用 `/root/.local/lib/node_modules/openclaw`，可评估是否归档源码树或改为浅克隆/构建产物保留。

7. **系统失败单元存在**
   - 证据：`fwupd-refresh.service`、`systemd-networkd-wait-online.service` failed。
   - 影响：大概率非核心业务，但会污染健康状态。
   - 建议：确认网络栈与 fwupd 是否需要；若不需要，记录后 mask/disable 需另行授权。

8. **Nginx 旧配置文件较多**
   - 证据：`/etc/nginx/sites-available/zmjjkkk.fun.bak.*`、`.disabled.*`、`.pre-fix-*` 多份。
   - 影响：配置排查时容易混淆；grep 输出噪声大。
   - 建议：保留当前 `zmjjkkk.fun` 与最近一次备份，其余移入归档目录。

### P2 / 低优先级/观察项

9. **OpenClaw 配置备份副本较多**
   - 证据：`/root/.openclaw/openclaw.json.*` 多份。
   - 建议：保留 `last-good` 和最近 2-3 份，其余归档。

10. **Codex 插件临时目录与缓存存在重复内容**
   - 证据：`/root/.codex/.tmp` 约 `86M`，重复文件扫描显示大量 `.tmp/plugins` 与 `plugins/cache` 重复。
   - 建议：确认无正在运行 Codex 任务后可清理 `.tmp`。

11. **证书均有效，但接近 90 天窗口**
   - 证据：多个证书有效期到 `2026-09-20`，certbot.timer 已存在。
   - 建议：保持 certbot.timer；后续可增加续期成功/失败提醒。

12. **Hermes System Monitor 后端已优化但备份散落**
   - 证据：`/opt/hermes-system-monitor` 内存在多份 `collector.py.bak.*` 和 `static/index.html.bak.*`。
   - 建议：把已验证的回滚点统一收纳，避免后续审计误判。

## 后续执行建议（仅建议，未执行）

1. 先处理 `hermes-gateway.service` 重启循环：确认 PID、unit、锁文件/运行目录状态，再决定 restart/replace/disable 策略。
2. 通过 OpenClaw schema 配置 `gateway.auth.token`，恢复 Gateway cron/QQBot approvals 授权链路。
3. 逐个确认公网端口用途，把 9000/9100/9200/9300/9500/9600/7000 尽量收敛到 Nginx 反代 + localhost 后端。
4. 建立 `/root/server-organization/archive/` 归档规则：Nginx 旧配置、OpenClaw config 旧备份、monitor 旧备份、Codex tmp。
5. 对 `/www/data/pnpm/store` 做只读依赖确认后，再考虑 prune。

## 未执行事项

- 未修改任何业务代码。
- 未删除任何文件。
- 未重启/停止任何服务。
- 未读取或输出密钥值，仅列出配置键名与文件路径。
- 未触碰 `sub2api/sub` 与 `upstream` 业务目录。

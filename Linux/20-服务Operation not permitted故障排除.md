## 问题现场
启动或重启服务后，进程没有启动，例如在执行 `systemctl start crond.server`后 crond 进程未启动，查询 `/var/log/messages` 得到以下信息：
```
May 18 19:53:28 3-Game-Server1 systemd[1]: Started Command Scheduler.
May 18 19:53:28 3-Game-Server1 systemd[2202718]: crond.service: Failed to adjust resource limit RLIMIT_NOFILE: Operation not permitted
May 18 19:53:28 3-Game-Server1 systemd[2202718]: crond.service: Failed at step LIMITS spawning /usr/sbin/crond: Operation not permitted
May 18 19:53:28 3-Game-Server1 systemd[1]: crond.service: Main process exited, code=exited, status=205/LIMITS
May 18 19:53:28 3-Game-Server1 systemd[1]: crond.service: Failed with result 'exit-code'.
```
根据关键错误信息`Failed to adjust resource limit RLIMIT_NOFILE`可以推测跟系统的打开文件数量限制有关。


## 解决过程

1. 查看系统 ulimit
    ```bash
    ulimit -n #显示 65535
    ```
2. 查看sysctl.conf
    ```bash
    sysctl -a |grep fs.nr_open #显示 fs.nr_open=65535
    ```
3. 查看系统已打开的文件数
    ```bash
    lsof -Ki |wc -l #并未超过65535
    ```

## 解决办法
编辑 `/usr/lib/systemd/system/crond.service`，修改 `[Service]` 部分的 `LimitNOFILE` 配置为 65535(不能超过系统最大打开文件限制数量)，如下：
```
[Service]
EnvironmentFile=-/etc/default/cron
ExecStart=/usr/sbin/cron -f -P $EXTRA_OPTS
IgnoreSIGPIPE=false
KillMode=process
Restart=on-failure
LimitNOFILE=65535
```
修改完后重新加载服务后启动服务。

```bash
systemctl daemon-reload
systemctl restart crond.service
```
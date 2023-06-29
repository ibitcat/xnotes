## 问题一
Filebeat 在运行一段时间后自动关闭，查询日志有以下报错：
```
"File is inactive. Closing because close_inactive of 5m0s reached."
```
或者
```
"Reader was closed. Closing."
```

### 原因
一段时间内被监测的文件没有新的输入 filebeat(close_inactive) 会自动关闭，默认值是5m。
当logback等系统滚动日志名称的时候，filebeat持有旧文件的句柄，close_inactive 时间过后会自动关闭，scan_frequency 是扫描新文件的频率。
使用nohup的方式启动的话会时常挂掉，而使用 service 方式启动，则可以在 filebeat 挂掉后自动重启。

可以修改配置，增大 close_inactive 的时长，例如：
```
filebeat.prospectors:
 - input_type: log
   close_inactive: 10m
```

### 解决方法
推荐使用 service 的方式启动 filebeat，新建 `/usr/lib/systemd/system/filebeat.service` 文件，输入以下内容：
```
[Unit]
Description=filebeat server daemon
Documentation=https://www.elastic.co/products/beats/filebeat
Wants=network-online.target
After=network-online.target

[Service]
#Environment="BEAT_LOG_OPTS=-e"
Environment="BEAT_CONFIG_OPTS=-c /var/local/filebeat/sensorsdata.yml"
Environment="BEAT_PATH_OPTS=-path.home /var/local/filebeat -path.config /var/local/filebeat -path.data /var/local/filebeat/data -path.logs /var/local/filebeat/logs"
WorkingDirectory=/var/local/filebeat
ExecStart=/var/local/filebeat/filebeat $BEAT_CONFIG_OPTS $BEAT_PATH_OPTS
Restart=always

[Install]
WantedBy=multi-user.target
```

然后创建软连接：
```
sudo ln -s /usr/lib/systemd/system/filebeat.service /etc/systemd/system/multi-user.target.wants/filebeat.service
```

最后 reload 并启动 filebeat 服务。
```
sudo systemctl daemon-reload
sudo systemctl start filebeat
```

需要**注意**的是，若服务中设置了 `"BEAT_LOG_OPTS=-e"`，那么 filebeat 的日志会由 journald 管理，是不会被写入到 `path.logs` 所指定的路径的。
参见官方文档：
```
Logs are stored by default in journald. To view the Logs, use journalctl:
journalctl -u filebeat.service
```

### 参考

- [Filebeat and systemd](https://www.elastic.co/guide/en/beats/filebeat/7.1/running-with-systemd.html#_customize_systemd_unit_for_filebeat)
- [ELKF安装及安装过程中的问题的解决](https://www.jianshu.com/p/6720fe6d24fb)
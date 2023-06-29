## 创建定时任务
创建 cron 定时任务有两种方式：

- 方式1：
输入 `crontab -e` 命令，在打开的文件中填入定时任务，默认使用 VI 作为编辑器。

- 方式2：
直接打开定时任务文件，每个用户都有一个定时任务文件，存放在 `/var/spool/cron` 目录下。例如 `/var/spool/cron/root` 表示是root用户的定时任务文件。

创建好的定时任务，可以使用 `crontab -l` 命令来查看。另外，需要注意是否启动了 `crond` 服务。

```shell
# 查询 crond 服务状态
systemctl status crond

# 启动 crond 服务
systemctl start crond

# 关闭 crond 服务
systemctl stop crond

# 设置开机启动
systemctl enable crond

# 停止开机启动
systemctl disable crond
```

## crontab 命令选项

- u 指定一个用户
- l 列出某个用户的任务计划
- r 删除某个用户的任务
- e 编辑某个用户的任务

`-u`选项一般root用户在执行这个命令的时候需要此参数，例如：`crontab -u root -l`， 表示root用户查看自己的cron设置。

## cron文件语法
用户crontab文件中的每一行包含六个字段，每个字段之间用空格隔开，后面跟着要运行的命令:
```
* * * * * <username> command(s)
```
系统级 crontab 文件的语法与用户级 crontab 略有不同。它包含一个附加的强制用户字段`<username>`，指定哪个用户将运行cron作业。

前5个字段定义了定时任务的触发时间，包括了分、时、天、月、周，它们的取值范围如下：
```
* * * * * command(s)
^ ^ ^ ^ ^
| | | | |     allowed values
| | | | |     -------
| | | | ----- Day of week (0 - 7) (Sunday=0 or 7)，每周的周几执行该任务
| | | ------- Month (1 - 12)，每年的第几个月执行该任务
| | --------- Day of month (1 - 31)，每月的第几天执行该任务
| ----------- Hour (0 - 23)，每天的第几个小时执行该任务
------------- Minute (0 - 59)，每个小时的第几分钟执行该任务
```

- `*`: 星号操作符表示所有允许的值。如果在分钟字段中有星号，则表示任务将每分钟执行一次。
- `-`: 连字符操作符允许您指定一个值范围。如果您在Day of the week字段中设置了1-5，任务将在每个工作日(从周一到周五)运行。范围是包含的，这意味着第一个值和最后一个值都包含在范围中。
- `,`: 逗号操作符允许您定义一个用于重复的值列表。例如，如果Hour字段中有1,3,5，任务将在1 am、3 am和5 am运行。列表可以包含单个值和范围，1-5、7、8、10-15
- `/`: 斜杠操作符允许您指定可以与范围一起使用的步骤值。例如，如果分钟字段中有1-10/2，这意味着在范围1-10中每两分钟执行一次操作，与指定1、3、5、7、9相同。您也可以使用星号操作符，而不是一组值。要指定每20分钟运行一次的作业，可以使用" */20 "。


## 环境变量

crontab 有一个坏毛病，就是它总是**不会缺省的从用户profile文件中读取环境变量参数**，经常导致在手工执行某个脚本时是成功的，但是到crontab中试图让它定期执行时就是会出错。这是因为 crontab 有自己的环境变量配置，在 `/etc/crontab` 文件中，并不会自动加载当前用户的环境变量，所以在执行命令之前，应该先配置好环境变量。

解决办法也很简单，就是在执行 commands 前，先加载用户的环境变量，例如：
```
*/2 * * * * source $HOME/.bashrc && bash /home/xxx/crontab.sh
```

## 运行日志

linux中默认情况下,crontab中执行的日志写在 `/var/log` 下。
```
# 显示 corn 任务执行日志
sudo ls /var/log/corn*
/var/log/cron /var/log/cron-20220815
```

另外，每条定时任务执行后，系统会给用户发送邮件，例如 `/var/mail/root`(或者`/var/spool/mail/root`) 表示 root 用户收到的系统邮件。

## Crontab 时区设置
时区设置不对，会导致定时任务执行不成功，例如设置每天12:00的任务，结果在每天20:00才启动。解决办法如下：
- 修改本地时区
    ```bash
    # 查看时区，例如显示 CST-8 表示东八区
    more /etc/localtime
    # 备份
    cp /etc/localtime /etc/localtime.bak
    # 设置本地时区
    cp -pf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    ```
- 修改 crontab 时区
    ```bash
    vim /etc/crontab
    # 添加变量 CRON_TZ=Asia/Shanghai
    ```
- 重启 crond 服务
    ```
    systemctl restart crond.service
    ```

## 示例
- 每隔5分钟执行一次任务
    ```
    */5 * * * * commands  #表示每分钟的第0、5、10、15、20、...55 执行一次
    ```

- 每个星期一的上午8点到11点的第3和第15分钟执行
    ```
    3,15 8-11 * * 1 commands
    ```

- 每个9分钟执行一次任务
    ```
    # 9不能被60整除，所以该任务并不是严格按照每9分钟执行一次，在第54分钟执行后，间隔6s进入下一分钟开始新的循环
    */9 * * * * commands  #表示每分钟的第0、9、18、27、36、45、54 执行一次
    ```

- 间隔N秒执行任务[^参考]

    crontab的间隔粒度是1分钟，若要实现秒粒度的定时任务，有两种方法：
    1. 利用 crontab 的延时

        通过延时方法 sleep N 来实现每N秒执行。例如每隔10秒执行一次任务：
        ```
        * * * * * sleep 10; commands
        * * * * * sleep 20; commands
        * * * * * sleep 30; commands
        * * * * * sleep 40; commands
        * * * * * sleep 50; commands
        ```
    2. shell 脚本实现
        编写一个脚本 `crontab.sh`，代码如下：
        ```shell
        #!/bin/bash
        step=2 #间隔的秒数，不能大于60
        for (( i = 0; i < 60; i=(i+step) )); do
            data >> /data/crontab.log
            sleep $step
        done
        exit 0
        ```
        然后，添加一个每分钟执行的定时任务,
        ```
        * * * * * bash /data/crontab.sh
        ```
        即可实现。


[^参考]:https://www.cnblogs.com/yuyanc/p/16434413.html
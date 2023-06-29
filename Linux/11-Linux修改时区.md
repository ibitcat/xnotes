## 查看当前时区
```
date -R
#输出： Fri, 26 Aug 2022 18:43:35 +0800
# 后面的 +0800 表示时区

# 或者
date +"%z"

# 查看当前使用的时区文件
ls -al /etc/localtime
```

## 修改系统时区
```bash
# -f 强制修改
sudo ln -s -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 修改用户时区
Linux 用户一个多用户系统，每个用户都可以配置自己所需的时区，可以在用户目录的 `.bashrc` 中新增一个 TZ 环境变量：
```
export TZ='Asia/Shanghai'
```
然后重新登陆或执行 `source .bashrc` 生效。


## timedatectl 设置时区
```
sudo timedatectl set-timezone 'Asia/Shanghai'
```

## 检查是否为夏令时
可以借助 lua，运行以下 lua 代码：
```lua
print(os.date("*t").isdst) -- 输出true 表示为夏令时
```

## 时区列表
维基百科[时区列表](https://zh.wikipedia.org/zh-cn/%E6%97%B6%E5%8C%BA%E5%88%97%E8%A1%A8)列出了列举世界各国和地区的法定时区，带`*`表示该地区启用了夏令时。
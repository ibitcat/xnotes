以安装 [思源黑体 Source Han Sans](https://github.com/adobe-fonts/source-han-sans) 为例。

注意，字体一般分为两种：
- Sans，无衬线版本
- Serif，有衬线版本

另外，字体文件推荐otf，其次ttc，最后ttf。

## 查看字体
```bash
# 查看所有已安装的字体
fc-list

# 查看已安装的中文字体
fc-list :lang=zh
```

## 安装字体
```bash
# ubuntu 22.04
# 下载字体压缩包
wget https://github.com/adobe-fonts/source-han-sans/releases/download/2.004R/SourceHanSansCN.zip

# 解压
unzip SourceHanSansCN.zip

cp -r SubsetOTF/CN/ /usr/share/fonts/SourceHanSans

# 安装字体
fc-cache -f -v
```

如果是 centos 系统，可能需要安装 `mkfontscale` 和 `fontconfig`，安装命令如下：
```
yum install mkfontscale
yum install fontconfig
cd /usr/share/fonts/SourceHanSans
mkfontscale
mkfontdir
fc-cache -f -v
```

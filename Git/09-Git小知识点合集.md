## 子模块
添加子模块的命令如下：
```
git submodule add -f 子模块git仓库地址 子模块路径

# 示例：
git submodule add -f -b main https://github.com/cloudwu/skynet.git skynet
git submodule add -f https://github.com/starwing/lua-protobuf.git 3rd/lua-protobuf
```

添加完后，在仓库的根目录会产生一个 `.gitmodules` 的文件，里面就是已添加的子模块列表。

小问题：
如果你要复制一个项目，如果只简单复制 `.gitmodules` 文件，那么在更新子模块是会报以下错误：
```
git submodule update --init --recursive -- "skynet"
error: pathspec 'skynet' did not match any file(s) known to git
```

解决办法：删除子目录文件夹，再添加一次子模块。
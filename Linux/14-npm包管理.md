## Node.js包管理工具
npm 是 node.js 的包管理工具，通常不需要单独安装，在安装Node.js时自带，用于安装Node.js的各种包或者依赖。

## 在线安装
npm 支持多种方式的安装，具体参见帮助文档(`npm help install`)， 下面以 `docsify-cli` 为例。
- 全局安装：`npm i docsify-cli --location=global`，包安装后的路径在 node 安装路径的 `lib/node_modules`，例如：`/usr/local/node/lib/node_modules`。
- 本地安装：`npm i docsify-cli`，包安装在当前目录的 `node_modules` 文件夹中。

## 离线安装
有两种方法，且都需要借助一个有网络的环境。本质都是得到一个 tgz 包，然后在离线环境下使用命令 `npm i xxx.tgz` 安装。

- 直接打包成 tgz 包(**注意**：被打包的文件夹要命名为 `package`)
    ```
    cd /usr/local/node/lib/node_modules
    cp -rf docsify-cli/ package
    tar -zcvf docsify-cli.tgz package
    rm -rf package
    ```
- 使用 `npm-pack-all` 工具
    ```
    npm install npm-pack-all --location=global
    cd /usr/local/node/lib/node_modules/docsify-cli
    npm-pack-all
    ```
推荐使用**第二种**方式。

## 参考
- [第三方npm包](http://cuketest.com/zh-cn/misc/npm/)
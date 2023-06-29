## txt 显示乱码

编辑 nginx.conf 文件，在 server 块里指定字符编码为utf-8，例如：
```
server {
        listen 80;
        charset 'utf-8';  #防止txt文本出现乱码，一定要加单引号

        location / {
            root   /data/www;
            index  index.html index.htm;
        }
}
```

## html 文件的css不生效
在部署 Nginx 服务器后，打开浏览器能够正常获取到 css 文件，但是 css 的效果却不生效。

按 F12 打开控制台，切换到 `Network`(网络) 标签，可以看到 css 文件能够正常请求到，但是该文件的响应标头中的 `content-type` 一栏中显示的是: `text/plain`。所以浏览器就把该 css 文件当作是一个纯文本文件显示了。

为了让 css 生效，需要在nginx 配置文件中的 http 块头部添加下面的配置内容：
```
http {
    include mime.types;
    default_type application/octet-stream;

    #...
}
```

mime type 和文件扩展名的对应关系一般放在 mime.types这个文件里，然后用 `include mime.types;`来加载，一般安装 Nginix 后都会在附带了一个 mine.types 文件，Nginx 会根据mime type定义的对应关系来告诉浏览器如何处理服务器传给浏览器的这个文件，是打开还是下载。

如果Web程序没设置，Nginx也没对应文件的扩展名，就用Nginx 里默认的 default_type 定义的处理方式，默认文件类型为 `application/octet-stream`。

## 打开目录浏览功能

Nginx默认是不允许列出整个目录的。如需此功能（文件服务器），在Nginx配置文件中的 server 或 location 段里添加上`autoindex on;`来启用目录浏览功能。例如：
```
server {
    listen 8080;
    charset 'utf-8';

    location / {
        root /data/docsify;
        index index.html index.htm;     #优先匹配主页html文件，若没有则直接浏览文件(如果想浏览index.html，把这行注释即可)
        autoindex on;                   #启用文件服务器
        autoindex_exact_size off;       #默认为on，显示出文件的确切大小，单位是bytes。改为off后，显示出文件的大概大小，单位是kB或者MB或者GB
        autoindex_localtime on;         #默认为off，显示的文件时间为GMT时间。改为on后，显示的文件时间为文件的服务器时间
    }
}
```
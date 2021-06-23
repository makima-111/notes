# h2o:web服务器

## 概述

H2O 是新一代 HTTP 服务器，与老一代 Web 服务器相比，它以更少的 CPU 使用率为用户提供更快的响应。 该服务器从头开始设计，充分利用 HTTP/2 功能，包括优先内容服务和服务器推送。



## 配置示例

```shell
listen: 19090
hosts:
  "127.0.0.1:19090":
    paths:
      /logLoad:
        file.dir: /data/logs/
        file.dirlisting: on
        gzip: on
        file.mime.addtypes:
          "text/plain":
            extensions: [".log"]
            is_compressible: yes

access-log: /var/log/h2o/access.log
error-log: /var/log/h2o/error.log
pid-file: /run/h2o/h2o.pid
```


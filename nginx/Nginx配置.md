# Nginx配置

## 一、配置

```bash
#登录服务器
41
# 切换至root
sudo su -
# 配置未见为nginx.conf,一般都会在其中音容conf.d中所有配置文件
# 切换至etc/nginx/cont
cd /etc/nginx/conf.d
# 创建配置文件
server {
	listen 9001;
	listen localhost;
	location ~ /edu/ {
		proxy_pass http://127.0.0.1:8080/;
	}
	location ~ /vod/ {
		proxy_pass http://127.0.0.1:8081/;
	}
}

# 刷新配置
nginx -sreload
```

### 二、日志

```shell
#记录post body
\t"$request_body"
```


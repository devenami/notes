### nginx 正则规则

以=开头表示精确匹配
如 A 中只匹配根目录结尾的请求，后面不能带任何字符串。
^~ 开头表示uri以某个常规字符串开头，不是正则匹配
~ 开头表示区分大小写的正则匹配;
~* 开头表示不区分大小写的正则匹配
/ 通用匹配, 如果没有其它匹配,任何请求都会匹配到

### 匹配顺序
  (location =) > (location 完整路径) > (location ^~) > (location ~) > (location ~*) > (location 部分路径) > (location /)

### 案例

```
	# 配置反向代理
	location / {
	   		rewrite '^/download/([\s\S]*)' /assetes/$1;
	        proxy_pass http://127.0.0.1:8080;
	        root html;
	        index index.html index.htm;
	}
	
	# 将所有以 /assetes开头的资源转发到 /opt/res/assetes/目录下
	location ^~ /assetes/ {
	     root   /opt/res/;
	     autoindex on;
	}
	
	# 静态资源(网页静态资源使用)
	# 忽略大小写，并且匹配后面的正则
	location ~* \.(htm|html|gif|jpg|jpeg|png|css|js|icon)$ {
	        root  /opt/res/web/;
	    autoindex off;
	}
```
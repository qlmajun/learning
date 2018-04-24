### 安装Nginx+Lua开发环境搭建 ###
----

### 安装依赖

centos:

```
yum install pcre-devel openssl-devel gcc curl
```

Debian 和 Ubuntu 用户:

```
apt-get install libpcre3-dev libssl-dev perl make build-essential curl
```

### 下载,解压ngx_openrestry

下载，解压ngx_openresty-1.7.7.2.tar.gz到/usr/servers目录下

```
wget http://openresty.org/download/ngx_openresty-1.7.7.2.tar.gz  
tar -xzvf ngx_openresty-1.7.7.2.tar.gz  
```


**ngx_openresty-1.7.7.2/bundle目录里存放着nginx核心和很多第三方模块，比如有我们需要的Lua和LuaJIT。**


### 安装LuaJIT

```
cd bundle/LuaJIT-2.1-20150120/  
make clean && make && make install  
ln -sf luajit-2.1.0-alpha /usr/local/bin/luajit  
```

### 下载ngx_cache_purge模块，该模块用于清理nginx缓存

```
cd /usr/servers/ngx_openresty-1.7.7.2/bundle  
wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz  
tar -xvf v0.3.0.tar.gz   
```

### 安装ngx_openresty

```
cd /usr/servers/ngx_openresty-1.7.7.2  
./configure --prefix=/usr/servers --with-http_realip_module  --with-pcre  --with-luajit --add-module=./bundle/ngx_cache_purge-2.3/ --add-module=./bundle/nginx_upstream_check_module-0.3.0/ -j2  
make && make install
```

--with***                安装一些内置/集成的模块

--with-http_realip_module  取用户真实ip模块

-with-pcre               Perl兼容的达式模块

--with-luajit              集成luajit模块

--add-module            添加自定义的第三方模块，如此次的ngx_che_purge

### nginx+lua项目构建

项目结构如下：

example

&emsp;example.conf

&emsp;lua

&emsp;&emsp; test.lua

&emsp;lualib

&emsp;&emsp; \*.so

&emsp;&emsp; \*.lua

### 配置环境

* 1、编辑nginx.conf 在http部分添加如下配置

```
#lua模块路径，多个之间”;”分隔，其中”;;”表示默认搜索路径，默认到/usr/servers/nginx下找  
lua_package_path "/usr/servers/lualib/?.lua;;";  #lua 模块  
lua_package_cpath "/usr/servers/lualib/?.so;;";  #c模块
include /usr/example/example.conf;
```

* 2、编辑example.conf

```
server {  
    listen       80;  
    server_name  _;  

    location /lua {  
        default_type 'text/html';  
        lua_code_cache off;  
        content_by_lua_file /usr/example/lua/test.lua;  
    }  
}  
```

* 3、编辑test.lua

```
ngx.say("hello world");  
```

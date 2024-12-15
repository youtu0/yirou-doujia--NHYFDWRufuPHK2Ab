
**需求1：某几个ip的代理**




```
server {
         listen 9000;    # 监听端口
         server_name localhost;
        
        set $url "proxy_server_doman_or_ip";    # 设置代理的域名或IP变量，这里替换成自己需要代理的网站
         location / {
             proxy_pass http://$url:8082;    # 将请求转发到由 $url 变量表示的地址。
         }
    }
```


**需求2：域名网站的代理**




```
server {
         listen 9000;    # 监听端口
         server_name localhost;

        # 设置DNS解析器的地址为8.8.8.8，并且设置了解析器的缓存时间为300秒（这样每隔300s就会重新解析一次）。ipv6=off 是关闭IPv6的解析支持。
        resolver 8.8.8.8 valid=300 ipv6=off; 
        resolver_timeout 3s;    # 设置解析DNS的超时时间为3秒
        
        proxy_read_timeout 60s;
        proxy_send_timeout 60s;
        proxy_connect_timeout 60s;
        
        set $url "proxy_server_doman_or_ip";    # 设置代理的域名或IP变量，这里替换成自己需要代理的网站
         location / {
            proxy_pass http://$url:9000;    # 将请求转发到由 $url 变量表示的地址。9000是目标网站的端口。
            
            proxy_buffers 256 4K;        # 设置用于缓存后端响应的缓冲区大小为256个，每个大小为4K。
            proxy_max_temp_file_size 0;        # 设置Nginx暂存响应数据的最大临时文件大小为0，即不使用临时文件。
            proxy_cache_valid 200 302 1m;     # 针对状态码为200和302的响应，设置缓存有效期为1分钟。
            proxy_cache_valid 301 1h;        # 针对状态码为301的响应，设置缓存有效期为1小时。
            proxy_cache_valid any 1m;    # 对于其他任何响应状态码，设置缓存有效期为1分钟。
         }
    }
```


**需求3：所有网站的代理**




```
server {
    # 服务器监听的端口号为8080
    listen                           8080;
 
    # 服务器名称为localhost
    server_name                      localhost;
 
    # 指定DNS服务器地址为114.114.114.114，禁用IPv6解析
    resolver                         114.114.114.114 ipv6=off;
 
    # 开启HTTP CONNECT方法支持，用于建立与后端服务器的TCP连接
    proxy_connect;
 
    # 允许通过代理的端口，这里允许443和80端口
    proxy_connect_allow              443 80;
 
    # 建立连接的超时时间为10秒
    proxy_connect_connect_timeout    10s;
 
    # 读取数据的超时时间为10秒
    proxy_connect_read_timeout       10s;
 
    location / {
        # 将请求转发到代理目标
        proxy_pass $scheme://$http_host$request_uri;
    }
```


代理验证：




```
 curl -I https://blog.csdn.net/ -v -x 127.0.0.1:8080
```


如图 出现"HTTP/1\.1 200 Connection Established" 表示代理服务器已经成功建立了连接


![](https://img2024.cnblogs.com/blog/237138/202412/237138-20241214221303042-156910098.png)


![](https://img2024.cnblogs.com/blog/237138/202412/237138-20241214221638987-704259463.png)


 安装nginx默认不支持https，需要额外添加模块[ngx\_http\_proxy\_connect\_module](https://github.com/chobits/ngx_http_proxy_connect_module/ "ngx_http_proxy_connect_module")。需确保模块和Nginx版本匹配。


![](https://img2024.cnblogs.com/blog/237138/202412/237138-20241214222126900-1867374671.png)


![](https://img2024.cnblogs.com/blog/237138/202412/237138-20241214222153857-1337485942.png)




```
#安装patch：
yum install patch -y

cd /root
wget http://nginx.org/download/nginx-1.20.2.tar.gz

#解压
tar xf nginx-1.20.2.tar.gz 
 
#进入nginx目录
cd nginx-1.20.2/
 
#使用patch命令导入补丁 注意路径是否一致 我是直接在根目录操作的
patch -p1 < /root/ngx_http_proxy_connect_module-0.0.2/patch/proxy_connect_rewrite_1018.patch
 
```


下面安装nginx




```
#安装编译工具和库 
yum install gcc cmake make cmake unzip ncurses-devel gcc gcc-c++ -y
 
#配置Nginx编译选项,使其在编译Nginx时包含ngx_http_proxy_connect_module-0.0.2模块
./configure --prefix=/usr/local/nginx --add-module=/root/ngx_http_proxy_connect_module-0.0.2
 
#编译和安装Nginx
make && make install
```


**windows服务器代理配置**


![](https://img2024.cnblogs.com/blog/237138/202412/237138-20241214222648830-1397902155.png)


linux代理服务器设置




```
vi /etc/profile
 
#编辑/etc/profile文件 在最后一行加入
 
export http_proxy=192.168.0.97:8080
export https_proxy=192.168.0.97:8080
export ftp_proxy=192.168.0.97:8080
 
#192.168.0.97:8080 为你的代理服务器ip和端口
```


使用source命令使其生效




```
source /etc/profile
```


 


 本博客参考[milou加速器](https://jiechuangmoxing.com)。转载请注明出处！

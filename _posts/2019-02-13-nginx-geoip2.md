

# geoip2限制国家地区访问
nginx的geoip2模块，支持免费的geolite2数据库，可以限制国家和地区的ip访问。

maxmind的ip产品提供了两种数据库，geoip2和geolite2.前者是收费版后者是免费版。根据官方提供的数据收费版的精确度比免费版高2%左右，且收费版更新频率更高。但即使是免费版在国家范围内的精确度也到达了98%.



### 接下来说一下nginx添加geoip2模块的流程。

#### 下载nginx源码

```text
wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

#### 下载geoip2模块

```text
git clone https://github.com/leev/ngx_http_geoip2_module.git
```

#### 编译动态模块geoip2

```text
cd nginx-1.16.1
./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-stream_ssl_preread_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-http_auth_request_module --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E' --add-dynamic-module=./ngx_http_geoip2_module
```

#### 获取动态模块

```text
cd ~/nginx-1.16.1/objs
ls
ngx_http_geoip2_module.so
```

#### nginx配置

/etc/nginx/nginx.conf

```text
load_module  /usr/lib64/nginx/modules/ngx_http_geoip2_module.so

http {

    geoip2 /usr/local/share/GeoIP/GeoLite2-Country.mmdb {

    $geoip2_data_country_code country iso_code;

  }

  map $geoip2_data_country_code $allowed_country {

    default no;

    CN yes;

  }
}
```

server字段里配置

```text
if ($allowed_country = no) {

      return 502;

    }
```

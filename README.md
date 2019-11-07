# php-alpine + nginx

### 说明

最近在折腾PHP和NGINX的镜像, 之前我们用ubuntu来作为基础镜像构建的php, 实在是大得有些令人难以接受, 去到1G了

基于 *nginx* `1.16.1` *php* `7.2.24` *alpine* `3.10` 制作的镜像

安装完扩展以后, 镜像大小大概是 **101MB** 左右

php使用memcheched的话必须依赖 **libmemcached** , 镜像大小会增加大约 **10M** 左右, 其实我在思考是否有必要禁止使用这么古老的缓存, 全部走 **redis** 就好了;

**ffmpeg** 后面也应该是独立的服务，倘若有需要在 **php** 里面调用 **ffmpeg**, 还不如直接独立。 安装 **ffmpeg** ,镜像大小会增加大约 **50MB** 左右

各位自己考虑下···怎么使用比较合适。后面逐步更新其他版本的镜像组合。欢迎各位大佬们指教···下班收工

### 配置路径

#### nginx
/etc/nginx/nginx.conf
/etc/nginx/conf.d/default.conf

#### php
/usr/local/etc/php/conf.d/docker-php-ext-mysqli.ini
/usr/local/etc/php/conf.d/docker-php-ext-opcache.ini
/usr/local/etc/php/conf.d/docker-php-ext-pdo_mysql.ini
/usr/local/etc/php/conf.d/docker-php-ext-sodium.ini

/usr/local/etc/php-fpm.d/www.conf

### 看效果

```
docker build -t zhengzijian/7.2-fpm-alpine-github .
```

```yaml
version: "2"
services:
  worker:
    image: zhengzijian/7.2-fpm-alpine-github:latest
    ports:
      - "7000:80"
    restart: always
```

# Nextcloud

Nextcloud个人云储存，通过镜像的组合扩展了以下功能：

- Redis数据库加速
- 定时执行Cron（待验证）
- Nginx反向代理
- 自签名SSL证书

## 使用端口

默认采用`80`和`443`端口，分别提供给`http`和`https`。  

## 部署

### Step 1. 配置密码

所有的密码配置在`db.env`文件内，需要补全该文件中缺少的所有环境变量，同时修改`NEXTCLOUD_TRUSTED_DOMAINS`，将之后用于访问Nextcloud的域名添加到此位置（存在多个则使用空格分隔）。

### Step 2. 设置存储路径

默认的存储全部使用Docker提供的卷，卷删除之后数据随即丢失。  
如果需要指定数据存储路径，需在`docker-compose.yml`修改以下的卷出现的所有位置。

| 卷 | 作用 |
| ---| ---|
| `db` | 数据库文件 |
| `nextcloud`  | Nextcloud数据文件|

> :warning: 一定要保证Docker对指定的存储路径有读写权限，否则容器会不断重启（`docker ps`可以查看到）。

### Step 3. 运行部署

```shell
cd nextcloud
docker-compose up -d --build
```

## HTTPS

如需要启用HTTPS，则在使用`docker-compose up -d --build`启动之前，再做以下配置。

### Step 1. 修改域名

替换以下文件中的`www.sample.com`为自身Server的域名：`certbot/docker-compose.yml`、`nextcloud/nginx/nginx.conf`。

同时，取消`nextcloud/nginx/nginx.conf`中以下内容的注释，使配置生效。

```conf
        listen 443 ssl;
        listen [::]:443 ssl ipv6only=on;

        ssl_certificate /etc/letsencrypt/live/www.sample.com/fullchain.pem; <!>
        ssl_certificate_key /etc/letsencrypt/live/www.sample.com/privkey.pem; <!>
        ssl_trusted_certificate /etc/letsencrypt/live/www.sample.com/chain.pem; <!>
```

### Step 2. 生成证书

运行cerbot，生成相应的证书。

```shell
cd certbot
docker-compose up -d --build

# 等certbot_main这一docker容器退出后运行
docker-compose down
```

此时`certbot/certs/live/<你的域名>`目录下应当包含新生成的证书。否则检查certbot_main的日志寻找问题。

之后启动即可。

```shell
cd nextcloud
docker-compose up -d --build
```

# DevDock

非常感谢 [laradock](https://github.com/LaraDock/laradock)， [DevDock](https://github.com/RystLee/DevDock) 是简化定制之后的产物，方便学习使用。

>构建可能很耗费时间，不需要定制的话，可以使用我build好的images :  https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=rystlee&starCount=0

## 支持的软件 (镜像)

- **数据库引擎**
    - Mysql
- **Mysql 管理工具**
    - phpmyadmin
- **缓存引擎:**
    - Redis
    - Memcached
- **搜索引擎**
    - elasticsearch
- **PHP 服务器**
    - Nginx
- **PHP 编译工具**
    - php-fpm (php7.1)
- **工具:**
    - Workspace (PHP7-CLI, SOAP, xDebug, Composer, Git, Node, YARN, Gulp, SQLite, Vim, Nano, cURL...)
>如果你找不到你需要的软件，构建它然后把它添加到这个列表。


## 安装

克隆 `DevDock` 仓库:

在你系统的任意位置（建议在你的工作目录）：

```bash
git clone https://github.com/RystLee/DevDock.git
cd DevDock
cp .env.example .env
```

> 查看 .env 文件你会发现很多环境配置项，在这里可以自行配置开发环境。

## 启动
进入到 DevDock 目录中

例如 Nginx 和 Mysql:

查看 docker-compose.yml 文件：

```yml
    applications:
        image: tianon/true
        volumes:
            - ${APPLICATION}:/var/www
```

DevDock 默认将同级目录下的所有文件映射到数据卷容器 applications 中。其实可以你完全可以灵活配置，添加多个映射，例如：

```yml
    volumes:
        - ../project1:/var/www
        - ../../project2:/var/www
```

创建网站配置文件 参考 nginx/sites/default.conf （**不要使用 default.conf，它会在容器中被删除**）

```conf
server_name laravel.dev;

root /var/www/laravel/public;
```

创建初始数据库信息，在 docker-compose.yml 文件中 （多个数据库请通过 phpmyadmin 或 手动进入到 mysql 容器中创建）：

```yml
    environment:
        MYSQL_DATABASE: homestead
        MYSQL_USER: homestead
        MYSQL_PASSWORD: secret
        MYSQL_ROOT_PASSWORD: root
```


然后运行：

`docker-compose up -d  nginx mysql`

你可以从以下列表选择你自己的容器组合：

nginx, php-fpm, mysql, redis, memcached, elasticsearch, workspace

将配置文件中的各种服务的 host 改为相应的**容器名称**，如：DB_HOST: mysql

说明： workspace 和 php-fpm 将运行在大部分实例中, 所有不用在命令中 up 加上它们.


进入 Workspace 容器, 执行像 (Artisan, Composer, Gulp, ...)等命令

`docker-compose exec -it -u devdock workspace bash`

增加 --user=devdock (例如 docker-compose exec --user=devdock workspace bash) 作为您的主机的用户创建的文件. (你可以从 docker-compose.yml 修改 PUID (User id) 和 PGID (group id) 值 )。


## 使用

### 灵活配置开发环境

在 docker-compose.yml 中，引用了很多环境变量，可自行在 .env 进行配置。典型的，我已经将 nginx 目录下 的 sites 目录映射到 nginx 容器，所以当你修改 nginx 网站配置文件后，只要重启 nginx 容器即可：

`docker-compose restart nginx`

### 常用命令

* 列出正在运行的所有容器

`docker ps`

* 你也可以使用以下命令查看当前 DevDock 启动的容器

`docker-compose ps`

* 关闭所有容器

`docker-compose stop`

* 停止某个容器:

`docker-compose stop {container_name}`

* 删除服务容器

`docker-compose down {container_name}`

    - 该命令不会删除你的数据卷容器，如果你重新创建服务容器，服务容器默认仍会使用上次创建的数据卷容器
     * 如果不加 {容器名称} ，命令会删除所有服务容器。

* 列出所有数据卷容器

 `docker volume ls` 

* 删除数据卷容器

 `docker volume rm <VOLUME NAME>`

* 删除所有数据卷容器

 `docker volume rm $(docker volume ls -q)`

* 删除所有未被使用的数据卷容器

 `docker volume rm $(docker volume ls -qf dangling=true)`

* 查看容器日志

Nginx 的日志在 logs/nginx 目录
查看其它容器日志 (Mysql, php-fpm, …) 你可以运行:

 `docker-compose logs {image-name}`

### 编辑 Docker 镜像

1. 找到你想修改的镜像的 `dockerfile` , 例如： `mysql` 在 `mysql/Dockerfile`.
2. 按你所要的编辑文件.
3. 重新构建镜像:

如果你做任何改变 Dockerfile 确保你运行这个命令,可以让所有修改更改生效:

`docker-compose build`

选择你可以指定哪个镜像 (而不是重建所有的镜像):

`docker-compose build {image-name}`


如果你想重新创建整个镜像，你需要使用 --no-cache 选项  

`docker-compose build --no-cache {container-name}`


### 增加更多镜像

为了增加镜像（软件）, 编辑 docker-compose.yml 添加容器细节， 你需要熟悉 [docker compose 文件语法](https://docs.docker.com/compose/compose-file/).

### 使用 Redis

* `docker-compose up -d redis`


- 以 Laravel 为例，打开 .env 文件，然后配置 REDIS_HOST

```env
...
REDIS_HOST=redis
...
```

如果在你的 .env 文件没有找到 REDIS_HOST 变量。打开数据库配置文件 config/database.php 然后用 redis 替换默认 IP 127.0.0.1 ，例如：


```php
'redis' => [
    'cluster' => false,
    'default' => [
        'host'     => 'redis',
        'port'     => 6379,
        'database' => 0,
    ],
],
```


- 启用Redis缓存或者开启Session管理也在`.env`文件中用`redis`替换默认`file`设置`CACHE_DRIVER` 和 `SESSION_DRIVER` 

```env
CACHE_DRIVER=redis
SESSION_DRIVER=redis
```


- Compose 安装 “predis/predis”:

```bash
composer require predis/predis:^1.0
```


- 你可以用以下代码在 Laravel 中手动测试：

```php
\Cache::store('redis')->put('DevDock', 'Awesome', 10);
```


### 使用 PHP

* PHP 扩展

PHP 的扩展 FPM 和 CLI 分别安装在 php-fpm 和 workspace 镜像当中，如果需要定制，请分别到 php-fpm/Dockerfile-xx 和 workspace/Dockerfile 文件中编辑。

* 选择 php-fpm 版本

默认运行 **php-fpm 7.1** 版本

切换版本 PHP 7.0 或 PHP 5.6：

    1. 打开 docker-compose.yml。
    2. 在PHP容器的 Dockerfile-71 文件。
    3. 修改版本号, 用 Dockerfile-56 或 Dockerfile-70 替换 Dockerfile-71
    4. 最后重建PHP容器 `docker-compose build php-fpm`

> 更多关于 PHP 基础镜像, 请访问 [PHP Docker官方镜像](https://hub.docker.com/_/php/).


* 修改 php-cli 版本

默认运行**php-cli 7.1**版本

> 说明: php-cli 只用于执行 Artisan 和 Composer 等命令，不服务于你的应用代码，这是 php-fpm 的工作，所以编辑php-cli 的版本不是很重要。

php-cli 安装在 workspace 镜像，改变 php-cli 版本你需要编辑 workspace/Dockerfile.

### 使用自定义域名

假定你的自定义域名是 laravel.dev

- 打开 /etc/hosts 文件，映射 laravel.dev 到 127.0.0.1
```bash
127.0.0.1    laravel.dev
```

- 打开你的浏览器访问 `http://laravel.dev`


你可以在 nginx 配置文件自定义服务器名称，如下：

```conf
server_name laravel.dev;
```


### 使用 Elasticsearch

进入到 elasticsearch 目录下，config 和 plugins 分别放置了配置文件和插件，可根据需要修改和添加，完成之后重建镜像

`docker-compose build elasticsearch`

### 安装全局 Composer 命令

为启用全局 Composer Install 在容器构建中允许你安装 composer 的依赖，然后构建完成后就是可用的。


- 打开 docker-compose.yml 文件

- 在 workspace 项中找到 COMPOSER_GLOBAL_INSTALL 选项并设置为 true

- 重建容器 `docker-compose build workspace`


### 安装 Node + NVM

在 workspace 容器安装 NVM 和 Nodejs

- 打开 `docker-compose.yml` 文件

- 在 workspace 项中找到 INSTALL_NODE 选项设为 true

- 重建容器 `docker-compose build workspace`


### 安装 xDebug

- 打开 `docker-compose.yml` 文件

- 在 php-fpm 和 workspace 项中分别找到 INSTALL_NODE 选项设为 true

- 重建容器 `docker-compose build workspace php-fpm`


## Debug

* 看到包含 address already in use 的错误：

确保你想运行的服务端口 (80, 3306, etc.) 不是已经被其他程序使用，例如 apache/httpd 服务或其他安装的开发工具

* 容器报类似 Connection refused 的错误：

将配置文件中服务的 host 改为相应的容器名称，如 mysql

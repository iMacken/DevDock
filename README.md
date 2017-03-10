# DevDock

非常感谢 [laradock](https://github.com/LaraDock/laradock)，通过这个基于Docker的开发环境包，我学到了很多，然后根据自己的需要删除了一些我认为不常用的部分、修改了部分配置以及增加了Elasticsearch容器，最终新开了自己的仓库 [DevDock](https://github.com/RystLee/DevDock)。另起炉灶是为了简化，方便学习。


### 支持的软件 (容器)

- **数据库引擎:**
	- MySQL
- **缓存引擎:**
	- Redis
	- Memcached
- **PHP 服务器:**
	- NGINX
- **PHP 编译工具:**
	- PHP-FPM (php5.6,php7.0,php7.1)
- **工具:**
	- Workspace (PHP7-CLI, SOAP, xDebug, Composer, Git, Node, YARN, Gulp, SQLite, Vim, Nano, cURL...)
>如果你找不到你需要的软件，构建它然后把它添加到这个列表。


## 安装

1 - 克隆 `DevDock` 仓库:

在你系统的任意位置（当然为了方便起见，进入到你的应用的父级目录）：

```bash
git clone https://github.com/RystLee/DevDock.git
```


## 使用

<br>
<br>

1 - 运行容器: *(在运行`docker-compose`命令之前，确认你在 `DevDock` 目录中*

**例子:** 运行 NGINX 和 MySQL:
运行之前：
查看 docker-compose.yml 文件：
```bash
    applications:
        image: tianon/true
        volumes:
            - ../:/var/www
```
这里将 `DevDock` 同级目录下的所有文件映射到数据卷容器 applications 中。其实可以你完全可以灵活配置，添加多个映射，例如：
```bash
    volumes:
        - ../project1:/var/www
        - ../../project2:/var/www
```

创建网站配置文件 参考 nginx/sites/default.conf （不要使用 default.conf，它会在容器中被删除）
例如：
```bash
server_name laravel.dev;

root /var/www/laravel/public;
```

创建初始数据库信息，在 docker-compose.yml 文件中：
```bash
    environment:
        MYSQL_DATABASE: homestead
        MYSQL_USER: homestead
        MYSQL_PASSWORD: secret
        MYSQL_ROOT_PASSWORD: root
```
根据需要进行修改即可。

然后运行：
```bash
docker-compose up -d  nginx mysql
```

你可以从以下列表选择你自己的容器组合：
`nginx`, `php-fpm`, `mysql`, `redis`, `memcached`, `elasticsearch`, `workspace`

将配置文件中的各种服务的 host 改为相应的容器名称，如：DB_HOST: mysql


**说明**: `workspace` 和 `php-fpm` 将运行在大部分实例中, 所有不用在命令中 `up`加上它们.



<br>
2 - 进入 Workspace 容器, 执行像 (Artisan, Composer, Gulp, ...)等命令

```bash
docker-compose exec -it workspace bash
```
<br />
增加 `--user=devdock` (例如 `docker-compose exec --user=devdock workspace bash`) 作为您的主机的用户创建的文件. (你可以从 `docker-compose.yml`修改 PUID (User id) 和 PGID (group id) 值 ).


<br>



## 文档

### [Docker]


### 列出正在运行的容器
```bash
docker ps
```
你也可以使用以下命令查看某项目的容器


```bash
docker-compose ps
```


<br>
### 关闭所有容器
```bash
docker-compose stop
```

停止某个容器:

```bash
docker-compose stop {容器名称}
```

<br>
### 删除服务容器
```bash
docker-compose down {容器名称}
```

该命令不会删除你的数据卷容器，如果你重新创建服务容器，服务容器默认仍会使用上次创建的数据卷容器

如果不加 {容器名称} ，命令会删除所有服务容器。

<br>
### 删除数据卷容器
使用 `docker volume ls` 列出所有数据卷容器，执行 `docker volume rm <VOLUME NAME>` 来

{Tip} 

1. 删除所有数据卷容器 `docker volume rm $(docker volume ls -q)`

2. 删除所有不被连接的数据卷容器 `docker volume rm $(docker volume ls -qf dangling=true)`


<br>
### 编辑Docker镜像

1 - 找到你想修改的镜像的 `dockerfile` ,
<br>
例如： `mysql` 在 `mysql/Dockerfile`.

2 - 按你所要的编辑文件.

3 - 重新构建容器:

如果你做任何改变`dockerfile`确保你运行这个命令,可以让所有修改更改生效:

```bash
docker-compose build
```

选择你可以指定哪个容器重建(而不是重建所有的容器):

```bash
docker-compose build {container-name}
```

如果你想重建整个容器，你可能需要使用 `--no-cache` 选项  (`docker-compose build --no-cache {container-name}`).


<br>
### 增加更多软件 (Docker 镜像)

为了增加镜像（软件）, 编辑 `docker-compose.yml` 添加容器细节， 你需要熟悉 [docker compose 文件语法](https://docs.docker.com/compose/compose-file/).



<br>
### 查看日志文件
Nginx的日志在 `logs/nginx` 目录

然后查看其它容器日志(MySQL, PHP-FPM,...) 你可以运行:

```bash
docker logs {container-name}
```


<br>
### 使用 Redis
1 - 首先务必用 `docker-compose up` 命令运行 (`redis`)容器.


```bash
docker-compose up -d redis
```


- 以Laravel为例，打开 `.env` 文件 然后 配置`redis`的`REDIS_HOST`

```env
REDIS_HOST=redis
```
如果在你的`.env` 文件没有找到`REDIS_HOST`变量。打开数据库配置文件`config/database.php`然后用`redis`替换默认IP`127.0.0.1`，例如：


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


- 最好务必通过Compose安装 `predis/predis` 包 `(~1.0)`:

```bash
composer require predis/predis:^1.0
```


- 你可以用以下代码在Laravel中手动测试：

```php
\Cache::store('redis')->put('DevDock', 'Awesome', 10);
```





<br>

### [PHP]


### 安装PHP扩展
安装PHP扩展之前,你必须决定你是否需要`FPM`或`CLI`,因为他们安装在不同的容器上,如果你需要两者,则必须编辑两个容器。

PHP-FPM拓展务必安装在 `php-fpm/Dockerfile-XX`. *(用你PHP版本号替换 XX)*.
<br>
PHP-CLI拓展应该安装到`workspace/Dockerfile`.


<br>
### 修改PHP-FPM版本
默认运行**PHP-FPM 7.1**版本.

#### 切换版本 PHP `7.1` 或 PHP `5.6`

1 - 打开 `docker-compose.yml`。

2 - 在PHP容器的 `Dockerfile-71`文件。

3 - 修改版本号, 用`Dockerfile-56` 或 `Dockerfile-70` 替换 `Dockerfile-71`

4 - 最后重建PHP容器

```bash
docker-compose build php
```

> 更多关于PHP基础镜像, 请访问 [PHP Docker官方镜像](https://hub.docker.com/_/php/).


<br>
### 修改 PHP-CLI 版本
默认运行**PHP-CLI 7.0**版本

>说明: PHP-CLI只用于执行Artisan和Composer命令，不服务于你的应用代码，这是PHP-FPM的工作，所以编辑PHP-CLI的版本不是很重要。
PHP-CLI安装在Workspace容器，改变PHP-CLI版本你需要编辑`workspace/Dockerfile`.
现在你必须手动修改PHP-FPM的`Dockerfile`或者创建一个新的。



<br>

### 使用自定义域名

假定你的自定义域名是 `laravel.dev`

- 打开 `/etc/hosts` 文件，映射 laravel.dev 到 127.0.0.1
```bash
127.0.0.1    laravel.dev
```

- 打开你的浏览器访问 `http://laravel.dev`


你可以在nginx配置文件自定义服务器名称,如下:

```conf
server_name laravel.dev;
```

<br>

### 灵活配置 nignx
在 docker-compose.yml 中，我已经将 sites 目录映射到 nginx 容器，所以当你修改 nginx 网站配置文件后，只要重启 nginx 容器即可：
```bash
docker-compose restart nginx
```

### 安装全局Composer命令

为启用全局Composer Install在容器构建中允许你安装composer的依赖，然后构建完成后就是可用的。


- 打开 `docker-compose.yml` 文件

- 在Workspace容器找到 `COMPOSER_GLOBAL_INSTALL` 选项并设置为 `true`


例如:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - COMPOSER_GLOBAL_INSTALL=true
    ...
```

- 现在特价你的依赖关系到 `workspace/composer.json`

- 重建Workspace容器 `docker-compose build workspace`



<br>
### 安装 Node + NVM

在Workspace 容器安装 NVM 和 NodeJS

- 打开 `docker-compose.yml` 文件

- 在Workspace容器找到 `INSTALL_NODE` 选项设为 `true`

例如:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - INSTALL_NODE=true
    ...
```


- 重建容器 `docker-compose build workspace`


### 安装 xDebug

1 - 首先在Workspace和PHP-FPM容器安装 `xDebug`:
<br>
a) 打开 `docker-compose.yml` 文件
<br>
b) 在Workspace容器中找到 `INSTALL_XDEBUG` 选项
<br>
c) 改为 `true`
<br>
d) 在PHP-FPM容器中找到 `INSTALL_XDEBUG ` 选项<br>
e) 改为 `true`

例如:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - INSTALL_XDEBUG=true
    ...
    php-fpm:
        build:
            context: ./php-fpm
            args:
                - INSTALL_XDEBUG=true
    ...
```

2 - 重建容器 `docker-compose build workspace php-fpm`



### DEBUG
#### 看到包含 `address already in use` 的错误
确保你想运行的服务端口(80, 3306, etc.)不是已经被其他程序使用，例如`apache`/`httpd`服务或其他安装的开发工具
#### mysql 等容器报 Connection refused 错误
将配置文件中数据库服务的 host 改为相应的容器名称 mysql


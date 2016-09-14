# DevDock

非常感谢[laradock](https://github.com/LaraDock/laradock)，通过这个基于Docker的开发环境包，我学到了很多，然后根据自己的需要删除了一些我认为不常用的部分、修改了部分配置以及增加了Elasticsearch容器，最终新开了自己的仓库[DevDock](https://github.com/RystLee/DevDock)。当然，推荐使用原仓库，我另起炉灶只是为了方便学习。


### 支持的软件 (容器)

- **数据库引擎:**
	- MySQL
- **缓存引擎:**
	- Redis
	- Memcached
- **PHP 服务器:**
	- NGINX
- **PHP 编译工具:**
	- PHP-FPM
- **工具:**
	- Workspace (PHP7-CLI, Composer, Git, Node, Gulp, SQLite, Vim, Nano, cURL...)
>如果你找不到你需要的软件，构建它然后把它添加到这个列表。


## 安装

1 - 克隆 `DevDock` 仓库:

进入到你的应用的上级目录：

```bash
git clone https://github.com/RystLee/DevDock.git
```


## 使用

<br>
<br>

1 - 运行容器: *(在运行`docker-compose`命令之前，确认你在 `DevDock` 目录中*

**例子:** 运行 NGINX 和 MySQL:
运行之前，先编辑网站配置文件 nginx/sites/site1.conf：
例如：
```bash
server_name laravel.dev;

root /var/www/laravel/public;
```
然后运行：
```bash
docker-compose up -d  nginx mysql
```
你可以从以下列表选择你自己的容器组合：
`nginx`, `php-fpm`, `mysql`, `redis`, `memcached`, `elasticsearch`, `workspace`.


**说明**: `workspace` 和 `php-fpm` 将运行在大部分实例中, 所有不用在命令中 `up`加上它们.



<br>
2 - 进入 Workspace 容器, 执行像 (Artisan, Composer, Gulp, ...)等命令

```bash
docker-compose exec workspace bash
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
### 删除所有容器
```bash
docker-compose down
```


小心这个命令,因为它也会删除你的数据容器。(如果你想保留你的数据你应该在上述命令后列出容器名称删除每个容器本身)




<br>
### 编辑Docker镜像

1 - 找到你想修改的镜像的 `dockerfile` ,
<br>
例如： `mysql` 在 `mysql/Dockerfile`.

2 - 按你所要的编辑文件.

3 - 重新构建容器:

```bash
docker-compose build mysql
```



<br>
### 建立/重建容器

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
默认运行**PHP-FPM 7.0**版本.

#### 切换版本 PHP `7.0` 到 PHP `5.6`

1 - 打开 `docker-compose.yml`。

2 - 在PHP容器的 `Dockerfile-70`文件。

3 - 修改版本号, 用`Dockerfile-56`替换 `Dockerfile-70`

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

- 打开 `/etc/hosts` 文件 添加以下内容，映射你的localhost 地址 `192.168.99.100` 为 `laravel.dev` 域名
```bash
192.168.99.100    laravel.dev
```

- 打开你的浏览器访问 `{http://laravel.dev}`


你可以在nginx配置文件自定义服务器名称,如下:


```conf
server_name laravel.dev;
```

<br>
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

### DEBUG
#### 看到包含 `address already in use` 的错误
确保你想运行的服务端口(80, 3306, etc.)不是已经被其他程序使用，例如`apache`/`httpd`服务或其他安装的开发工具
#### mysql 等容器报 Connection refused 错误
请将相关配置文件中的 host 指定为相应的容器名称，如：host: mysql

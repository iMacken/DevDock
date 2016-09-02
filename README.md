# DevDock

非常感谢[laradock](https://github.com/LaraDock/laradock)，通过这个基于Docker的开发环境包，我学到了很多，然后根据自己的需要删除了一些我认为不常用的部分、修改了部分配置以及增加了Elasticsearch容器，最终新开了自己的仓库[DevDock](https://github.com/RystLee/DevDock)。当然，推荐使用原仓库，我另起炉灶只是为了方便学习。


### 支持的软件 (容器)

- **数据库引擎:**
	- MySQL
	- MongoDB
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

克隆 `DevDock` 仓库:

```bash
git clone https://github.com/LaraDock/laradock.git
```


## 使用

<br>
<br>
- 运行容器: *(在运行`docker-compose`命令之前，确认你在 `DevDock` 目录中*

**例子:** 运行 NGINX 和 MySQL:

```bash
docker-compose up -d  nginx mysql
```
你可以从以下列表选择你自己的容器组合：
`nginx`, `php-fpm`, `mysql`, `redis`, `mongo`, `apache2`, `memcached`, `elasticsearch`, `workspace`.


**说明**: `workspace` 和 `php-fpm` 将运行在大部分实例中, 所有不用在命令中 `up`加上它们.



<br>
- 进入 Workspace 容器, 执行像 (Artisan, Composer, PHPUnit, Gulp, ...)等命令

```bash
docker-compose exec workspace bash
```
<br />
增加 `--user=laradock` (例如 `docker-compose exec --user=laradock workspace bash`) 作为您的主机的用户创建的文件. (你可以从 `docker-compose.yml`修改 PUID (User id) 和 PGID (group id) 值 ).


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

- 找到你想修改的镜像的 `dockerfile` ,
<br>
例如： `mysql` 在 `mysql/Dockerfile`.

- 按你所要的编辑文件.

- 重新构建容器:

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
- 首先务必用 `docker-compose up` 命令运行 (`redis`)容器.


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
\Cache::store('redis')->put('LaraDock', 'Awesome', 10);
```





<br>
### 使用 Mongo

- 首先在Workspace和PHP-FPM容器中安装`mongo`:
<br>
a) 打开 `docker-compose.yml` 文件
<br>
b) 在Workspace容器中找到`INSTALL_MONGO`选项：

<br>
c) 设置为 `true`
<br>
d) 在PHP-FPM容器中找到`INSTALL_MONGO` <br>
e) 设置为 `true`

相关配置项如下:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - INSTALL_MONGO=true
    ...
    php-fpm:
        build:
            context: ./php-fpm
            args:
                - INSTALL_MONGO=true
    ...
```

- 重建`Workspace、PHP-FPM`容器 `docker-compose build workspace php-fpm`



- 使用`docker-compose up` 命令运行MongoDB容器 (`mongo`)

```bash
docker-compose up -d mongo
```

在项目中配置，以laravel为例：

- 在`config/database.php` 文件添加MongoDB的配置项:

```php
'connections' => [

    'mongodb' => [
        'driver'   => 'mongodb',
        'host'     => env('DB_HOST', 'localhost'),
        'port'     => env('DB_PORT', 27017),
        'database' => env('DB_DATABASE', 'database'),
        'username' => '',
        'password' => '',
        'options'  => [
            'database' => '',
        ]
    ],

	// ...

],
```

- 打开Laravel的 `.env` 文件 然后 更新以下字段:

- 设置 `DB_HOST` 为 `mongo`的主机IP.
- 设置 `DB_PORT` 为 `27017`.
- 设置 `DB_DATABASE` 为 `database`.


- 最后务必通过Composer安装`jenssegers/mongodb`包，添加服务提供者（Laravel Service Provider）


```bash
composer require jenssegers/mongodb
```

- 测试:

- 首先让你的模型继承Mongo的Eloquent Model. 查看 [文档](https://github.com/jenssegers/laravel-mongodb#eloquent).
- 进入Workspace容器.
- 迁移数据库 `php artisan migrate`.




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

- 打开 `docker-compose.yml`。

- 在PHP容器的 `Dockerfile-70`文件。

- 修改版本号, 用`Dockerfile-56`替换 `Dockerfile-70`

- 最后重建PHP容器

```bash
docker-compose build php
```

> 更多关于PHP基础镜像, 请访问 [PHP Docker官方镜像](https://hub.docker.com/_/php/).


<br>
### 修改 PHP-CLI 版本
默认运行**PHP-CLI 7.0**版本

>说明: PHP-CLI只用于执行Artisan和Composer命令，不服务于你的应用代码，这是PHP-FPM的工作，所以编辑PHP-CLI的版本不是很重要。
PHP-CLI安装在Workspace容器，改变PHP-CLI版本你需要编辑`workspace/Dockerfile`.
现在你必须手动修改PHP-FPM的`Dockerfile`或者创建一个新的。 (可以考虑贡献功能).




<br>
### 安装 xDebug

- 首先在Workspace和PHP-FPM容器安装 `xDebug`:
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

- 重建容器 `docker-compose build workspace php-fpm`



<br>
### [Misc]


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


#### 看到包含 `address already in use` 的错误
确保你想运行的服务端口(80, 3306, etc.)不是已经被其他程序使用，例如`apache`/`httpd`服务或其他安装的开发工具

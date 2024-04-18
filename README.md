# cakephp-docker
A Docker Compose setup for containerized CakePHP Applications based on [cakephp-docker](https://github.com/cwbit/cakephp-docker) repo by [@cwbit](https://github.com/cwbit)

This setup spools up the following containers

* **php-fpm** (php 8.2)
* **nginx**
* **Databases**
	* **mysql** (8.0)

Content

* [Quick Start](#quick-start)
* [Installation](#installation)
* [How to use `bin/cake`, `mysql` and other commandline utils now that you're using containers](#now-how-to-run-bincake-and-mysql)
* [OK, so what did the defaults set up?](#ok-so-what-did-the-defaults-set-up)
* [Troubleshooting](#troubleshooting)
  * [nginx open logs/access.log failed no such file or directory](#nginx-open-logsaccesslog-failed-no-such-file-or-directory)
  * [creating a CakePHP app](#creating-a-cakephp-app)

## Quick Start

Follow the steps:

1. Clone or download the ZIP file for this repo
2. Put your CakePHP app inside the `cakephp` folder (if not, [create a CakePHP app](#creating-a-cakephp-app)). 
Your folder structure should look like this:
	```
	    somefolder
	        docker
				docker-compose.yml
				.env
				...
	        cakephp
	            .. put your cake app in here ..
	```
3. Go to `docker-compose.yml` and delete the service you're not using, for example mysql
	```	
		services:
			myapp-mysql:
				image: 'mysql:8.0'
				container_name: myapp-mysql
				...
	```
4. Go to `Dockerfile` on the `php-fpm` folder and comment the database extension you don't require, postgresql extension is set for default
	```	
		#For PostgreSQL
        php8.2-pgsql \
        #For MySQL
        #php8.2-mysql php8.2-sqlite3 \
	```	
5. From commandline, `cd` into the `docker` directory and run 
	```
	docker-compose up
	```
6. Go to `localhost:8180` and you'll see your app.

## Installation

Next, **Update the Environment File**

Copy or Rename `docker/.env.sample` to `docker/.env`.
This is an environment file that your Docker Compose setup will look for automatically which gives us a great, simple way to store things like your mysql database credentials outside of the repo.

By default the file will contain the following

```
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=myapp
MYSQL_USER=myapp
MYSQL_PASSWORD=myapp
```

Docker Compose will automatically replace things like `${MYSQL_USER}` in the `docker-compose.yml` file with whatever corresponding variables it finds defined in `.env`

Lastly, **Find/Replace** `myapp` with the name of your app.

> **WHY?** by default the files are set to name the containers based on your app prefix. By default this is `myapp`.
> A find/replace on `myapp` is safe and will allow you to customize the names of the containers
>
> e.g. myapp-mysql, myapp-php-fpm, myapp-nginx, myapp-mailhog

**Build and Run your Containers**

```bash
cd /path/to/your/app/docker
docker-compose up
```

That's it. You can now access your CakePHP app at

`localhost:8180`

> **tip**: start docker-compose with `-d` to run (or re-run changed containers) in the background.
>
> `docker-compose up -d`

**Connecting to your database**

Also by default the first time you run the app it will create a `MySQL` database with the credentials you specified in your `.env` file (see above)

``` yaml
host : myapp-mysql
username : myapp
password : myapp
database : myapp
```

You can access your MySQL database (with your favorite GUI app) on

`localhost:8106`

Your `cakephp/config/app_local.php` file should be set to the following (it connects through the docker link)

```php
  'Datasources' => [
    'default' => [
      'host' => 'myapp-mysql',
      'port' => '3306',
      'username' => 'myapp',
      'password' => 'myapp',
      'database' => 'myapp',
    ],
```

To change these defaults edit the variables in the `docker/.env` file or tweak the `docker-compose.yml` file under `myapp-mysql`'s `environment` section.

## Now, how to run `bin/cake` and `mysql`

Now that you're running stuff in containers you need to access the code a little differently

You can run things like `composer` on your host, but if you want to run `bin/cake` or use MySQL from commandline you just need to connect into the appropriate container first

**access your php server**

```bash
docker exec -it myapp-php-fpm /bin/bash
```
> remember to replace `myapp` with whatever you really named the container

**access mysql cli**

```bash
docker exec -it myapp-mysql /usr/bin/mysql -u root -p myapp
```
> remember to replace `myapp` with whatever you really named the container and with your actual database name and user login


## OK, so what did the defaults set up?

There are 4 containers that I use all the time that will be spooled up automatically

### `myapp-nginx` - the web server

First we're creating an nginx server. The configuration is set based on the CakePHP suggestions for nginx and `myapp-nginx` will handle all the incoming requests from the client and forward them to the `myapp-php-fpm` server which is what actually runs your PHP code.

You can configure the **nginx server** by editing the `/nginx/nginx.conf` file

### `myapp-php-fpm` - the PHP processor

This container runs `php` (and it's extensions) needed for your CakePHP app

It automatically includes the following extensions

* `php8.2-intl` (required for CakePHP 4.0+)
* `php8.2-mbstring`

If you uncomment the mysql extension, then
* `php8.2-sqlite3` (required for DebugKit)
* `php8.2-mysql`

It also includes some php ini overrides (see `php-fpm\php-ini-overrides.ini`)

This container will (by default) look for your web app code in `../cakephp/` (relative to the `docker-compose` file).

You can configure what **PHP extensions** are loaded by editing `/php-fpm/Dockerfile`

You can configure **PHP overrides** by editing `/php-fpm/php-ini-overrides.ini`

### `myapp-mysql` - the database server

The first time you run the docker containers it will create a folder in your root structure called `mysql` (at the same level as your `docker` folder) and this is where it will store all your database data.

Since the data is stored on your host device you can bring the mysql container up and down or completely destroy and rebuild it without ever actually touching your data - it is "safely" stored on the host.

You can configure **MySQL overrides** by editing `/mysql/my.cnf`


## Troubleshooting

### nginx open logs/access.log failed no such file or directory

submitted by @jeroenvdv

`myapp-nginx | nginx: [emerg] open() "/var/www/myapp/logs/access.log" failed (2: No such file or directory)`

This is caused by not installing CakePHP completely and can be fixed by creating the logs folder in your `myapp/cakephp` folder.

If you are starting fresh and need to install cake using the container you just created then follow the next step, [Creating a CakePHP app](#creating-a-CakePHP-app).

### creating a CakePHP app

Run the install command but set the app name to `.` instead of `myapp`.

```bash
docker exec -it myapp-php-fpm /bin/bash
```
> remember to replace `myapp` with whatever you really named the container

and then, inside the container
```bash
composer create-project --prefer-dist cakephp/app:~4.0 .
```
Next, fix the database connection strings by following the steps in [Connecting to your database](#connecting-to-your-database) (above).

That's it. You should have lots of happy green checkmarks at `localhost:8180` or whatever you set nginx to respond to.

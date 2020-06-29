
# Docker Compose and WordPress

[![Build Status](https://travis-ci.org/urre/wordpress-nginx-docker-compose.svg?branch=master)](https://travis-ci.org/urre/wordpress-nginx-docker-compose)

[![Donate](https://img.shields.io/badge/Donation-green?logo=paypal&label=Paypal)](https://www.paypal.me/urbansanden/5)

Use WordPress locally with Docker using [Docker compose](https://docs.docker.com/compose/)

This setup comes shipped with:

+ A `Dockerfile` for extending a base image and using a custom [Docker image](https://github.com/urre/wordpress-nginx-docker-compose-image) with an [automated build on Docker Hub](https://cloud.docker.com/repository/docker/urre/wordpress-nginx-docker-compose-image)
+ Custom domain for example `myapp.local`
+ Custom nginx config in `./nginx`
+ Custom PHP `php.ini` config in `./config`
+ Volumes for `nginx`, `wordpress` and `mariadb`
+ [Bedrock](https://roots.io/bedrock/) - modern development tools, easier configuration, and an improved secured folder structure for WordPress
+ Composer
+ [WP-CLI](https://wp-cli.org/) - WP-CLI is the command-line interface for WordPress.
+ [PhpMyAdmin](https://www.phpmyadmin.net/) - free and open source administration tool for MySQL and MariaDB
	- PhpMyAdmin config in `./config`
+ CLI scripts
	- Create a self signed SSL certificate for using https
	- Trust certs in macOS System Keychain
	- Setup the local domain in your in `/etc/hosts`

## Setup

### Requirements

+ [Docker](https://www.docker.com/get-started)
+ Openssl for creatng the SSL cert. Install using Homebrew `brew install openssl`

### Setup environment variables

Easily set your own local domain, db settings and more. Start by creating `.env` files, like the examples below.

#### For Docker and the cli scripts

Copy `.env-example` in the project root to `.env` and edit your preferences.

Example:

```dotenv
IP=127.0.0.1
APP_NAME=myapp
DOMAIN="myapp.local"
DB_HOST=mysql
DB_NAME=myapp
DB_ROOT_PASSWORD=password
DB_TABLE_PREFIX=wp_

```

#### For WordPress

Copy `.env-example` in the `src` folder to `.env` and edit your preferences.

Use the following database settings:

```dotenv
DB_HOST=mysql
DB_NAME=myapp
DB_USER=root
DB_PASSWORD=password
```

### Create SSL cert

#### macOS and Linux

```shell
cd cli
sudo ./create-cert.sh
```

> Note: OpenSSL needs to be installed.

### Windows

Use the bat file in in `./cli/windows scripts/create-cert.bat`

### Trust the cert

#### Add to macOS Keychain

Chrome and Safari will trust the certs using this script.

> In Firefox: Select Advanced, Select the Encryption tab, Click View Certificates. Navigate to where you stored the certificate and click Open, Click Import.

```shell
cd cli
sudo ./trust-cert.sh
```

#### Windows

Follow the instructions in  `./cli/windows scripts/trust-cert.txt`

### Add the local domain in /etc/hosts

To be able to use for example `https://myapp.local` in our browser, we need to modify the `/etc/hosts` file on our local machine to point the custom domain name. The `/etc/hosts` file contains a mapping of IP addresses to URLs.

#### macOS and Linux

```shell
cd cli
./setup-hosts-file.sh
```

> The helper script can both add or remove a entry from /etc/hosts. First enter the domain name, then press "a" for add, or "r" to remove. Follow the instructions on the screen.

#### Windows

Follow the instructions in  `./cli/windows scripts/setup-hosts-file.txt`

## Install WordPress and Composer packages (plugins/themes)
> i ran this in the main folder( where docker-compose.yml)
```shell
docker-compose run composer install
```
> If you have Composer installed on your computer you can also use `cd src && composer install`

### Update WordPress Core and Composer packages (plugins/themes)
> i ran this in the main folder( where docker-compose.yml)
```shell
docker-compose run composer update
```

## Run

```shell
sudo docker-compose up -d
```

Docker Compose will start all the services for you:

```shell
Starting myapp-mysql    ... done
Starting myapp-composer ... done
Starting myapp-phpmyadmin ... done
Starting myapp-wordpress  ... done
Starting myapp-nginx      ... done
```

🚀 Open [https://myapp.local](https://myapp.local) in your browser

Again do this to keep it up:
```shell
sudo docker-compose up
```

## PhpMyAdmin

PhpMyAdmin comes installed as a service in docker-compose.

🚀 Open [http://127.0.0.1:8080/](http://127.0.0.1:8080/)  in your browser

## Notes:

When making changes to the Dockerfile, use:

```bash
docker-compose up -d --force-recreate --build
```

## Tools

#### wp-cli

```shell
docker exec -it myapp-wordpress bash
```

Login to the container

```shell
wp search-replace https://olddomain.com https://newdomain.com --allow-root
```

Run a wp-cli command like this

> You can use this command first after you've installed WordPress using Composer as the example above.

### Changelog

#### 2020-05-03
- Added nginx gzip compression
#### 2020-04-19
- Added Windows support for creating SSH cert, trusting it and setting up the host file entry. Thanks to [@styssi](https://github.com/styssi)
#### 2020-04-12
- Remove port number from `DB_HOST`. Generated database connection error in macOS Catalina. Thanks to [@nirvanadev](https://github.com/nirvanadev)
- Add missing ENV variable from mariadb Thanks to [@vonwa](https://github.com/vonwa)
#### 2020-03-26
- Added phpMyAdmin config.Thanks to [@titoffanton](https://github.com/titoffanton)
#### 2020-02-06
- Readme improvements. Explain `/etc/hosts` better
#### 2020-01-30
- Use `Entrypoint` command in Docker Compose to replace the domain name in the nginx config. Removing the need to manually edit the domain name in the nginx conf. Now using the `.env` value `DOMAIN`
- Added APP_NAME in `.env-example` Thanks to [@Dave3o3](https://github.com/Dave3o3)
#### 2020-01-11
- Added `.env` support for specifying your own app name, domain etc in Docker and cli scripts.
- Added phpMyAdmin. Visit [http://127.0.0.1:8080/](http://127.0.0.1:8080/)

#### 2019-08-02
- Added Linux support. Thanks to [@faysal-ishtiaq](https://github.com/faysal-ishtiaq).

***

### Useful Docker Commands

Login to the docker container

```shell
docker exec -it myapp-wordpress bash
```

Stop

```shell
docker-compose stop
```

Down (stop and remove)

```shell
docker-compose down
```

Cleanup

```shell
docker-compose rm -v
```

Recreate

```shell
docker-compose up -d --force-recreate
```

Rebuild docker container when Dockerfile has changed

```shell
docker-compose up -d --force-recreate --build
```

### sass integration

> 1. First-line will create package.json
> 2. install laravel-mix 
> 3. copy and paste the webpack file from the module folder.

```shell
npm init -y
npm install laravel-mix --save-dev
cp node_modules/laravel-mix/setup/webpack.mix.js ./
```

> Paste this in the package.json

```shell
"scripts": {
    "dev": "npm run development",
    "development": "cross-env NODE_ENV=development node_modules/webpack/bin/webpack.js --progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js",
    "watch": "npm run development -- --watch",
    "hot": "cross-env NODE_ENV=development node_modules/webpack-dev-server/bin/webpack-dev-server.js --inline --hot --config=node_modules/laravel-mix/setup/webpack.config.js",
    "prod": "npm run production",
    "production": "cross-env NODE_ENV=production node_modules/webpack/bin/webpack.js --no-progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js"
}
```

> Paste this in the package.json

```shell
"scripts": {
    "dev": "npm run development",
    "development": "cross-env NODE_ENV=development node_modules/webpack/bin/webpack.js --progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js",
    "watch": "npm run development -- --watch",
    "hot": "cross-env NODE_ENV=development node_modules/webpack-dev-server/bin/webpack-dev-server.js --inline --hot --config=node_modules/laravel-mix/setup/webpack.config.js",
    "prod": "npm run production",
    "production": "cross-env NODE_ENV=production node_modules/webpack/bin/webpack.js --no-progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js"
}
```

> install this
```shell
npm install cross-env --save-dev
```

> Do a npm install again
> Then run the watch
```shell
npm run watch
```

> Add following code to function.php
```shell
function load_css() {
    wp_register_style('app', get_template_directory_uri() . '/dist/app.css', array(), false, 'all' );
    wp_enqueue_style('app');
}
add_action( 'wp_enqueue_scripts', 'load_css' );

function load_js()
{
    wp_enqueue_script('jquery');
    wp_register_script('main', get_template_directory_uri() . '/dist/app.js', array( 'jquery' ), false, true);
    wp_enqueue_script('main');
}
add_action('wp_enqueue_scripts', 'load_js');
```





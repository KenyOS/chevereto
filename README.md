[cheveretourl]: https://chevereto.com/
[cheveretoinstaller]: https://chevereto.com/download/file/installer
[cheveretogetstarted]: https://chevereto.com/get-started
[php]: https://hub.docker.com/_/php
[![chevereto](http://chevereto.com/app/themes/v3/img/chevereto-blue.svg)][cheveretourl]

# zaywalker/chevereto - Chevereto Installer

[Chevereto][cheveretourl] is a powerful and fast image hosting script that allows you to create your very own full featured image hosting website in just minutes.

This offers to install with [installer.php][cheveretoinstaller] at [Chevereto Get Started][cheveretogetstarted] page on [php 7.4 and apache2][php].

## Supported tags

* `installer` - Using the [installer script][cheveretoinstaller]

## Connection to database

[Chevereto][cheveretourl] requires an Mysql database to store its information. You can use a [Mysql](https://hub.docker.com/_/mysql/) or [MariaDB](https://hub.docker.com/_/mariadb/) container to host this.

Information on connection to database is provided to container via environment variables explained above.

## Persistent storage

[Chevereto][cheveretourl] stores images uploaded by users in `/var/www/html/images` directory within the container.

In order to manage Chevereto, recommand to [mount](https://docs.docker.com/engine/tutorials/dockervolumes/#data-volumes) `/var/www/html` directory with your local directory that you don't lose your images if you relaunch/remove container.

## Max image size

By default, PHP allow a maximum file upload to be 2MB. You can change such behaviour by updating the `/var/www/html/.htaccess` in your container or mounted directory `/your_mount/.htaccess`.

> Note that by default, Chevereto set a file upload limit of 10MB, so after you modify your `.htaccess`, you should also update this settings in Chevereto settings page (available at CHEVERETO_URL/dashboard/settings/image-upload)

> Copy and paste below to end of `.htaccess` gives you 10G limit. 
```
php_value upload_max_filesize 10G
php_value post_max_size 10G
php_value memory_limit 1G
php_value max_execution_time 300
php_value max_input_time 300
```

## Real-ip behind reverse proxy

If you run chevereto behind reverse proxy, you need to modify `/var/www/html/app/settings.php` in your container or mounted directory `/your_mount/app/setting.php`. 

> Copy and paste below to end of `settings.php` will enable to show real ip.
```
// Use X-Forwarded-For HTTP Header to Get Visitor's Real IP Address
if ( isset( $_SERVER['HTTP_X_FORWARDED_FOR'] ) ) {
        $http_x_headers = explode( ',', $_SERVER['HTTP_X_FORWARDED_FOR'] );

        $_SERVER['REMOTE_ADDR'] = $http_x_headers[0];
}
```

## Example Usage

I recommend you to use [Docker-compose](https://docs.docker.com/compose/) / [Docker swarm](https://docs.docker.com/engine/swarm/) to launch Chevereto in conjunction with a MySQL database. A sample of docker-compose.yaml can be found below.

### Docker compose

```yaml
version: '2.1'
services:
    mysql-chevereto:
      container_name: chevereto-mysql
      image: mariadb:10.2
      restart: always
      volumes:
        - mysql-vol-1:/var/lib/mysql/
      environment:
        # change timezone of yours and other values
        - TZ=Asia/Seoul
        - MYSQL_ROOT_PASSWORD=your_mysql_root_password
        - MYSQL_DATABASE=your_mysql_chevereto_dbname
        - MYSQL_USER=your_mysql_chevereto_username
        - MYSQL_PASSWORD=your_mysql_chevereto_user_password
      networks:
        chevereto-network:
          ipv4_address: 172.23.1.20
          aliases:
            - mysql

    web-chevereto:
      container_name: chevereto-web
      image: zaywalker/chevereto:installer
      restart: always
      depends_on:
        - mysql-chevereto
      volumes:
        - /your_mount/html:/var/www/html
      environment:
        # change timezone of yours and other values
        - TZ=${TZ}
      networks:
        chevereto-network:
          ipv4_address: ${IPV4_NETWORK:-172.23.1}.10
          aliases:
            - web

volumes:
  mysql-vol-1:

networks:
  chevereto-network:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.23.1.0/24
```

Once `docker-compose.yaml` is ready, you can run

```bash
docker-compose up
```

To run the service

### Standalone

```bash
docker run --name chevereto-web -d \
    -v /your_mount:/var/www/html/images \
    zaywalker/chevereto:installer
```

## Install chevereto

When the container is up and hooked up with your proxy, you can install chevereto to visit at

```
http://your.chevereto.web/installer.php
```
And follow installation. the values are

* License key
     - Enter your purchased key or skip to use Chevereto-Free
* cPanel access
     - Your cPanel account. If you don't have, just skip
* Database Host
     - Your MySQL server host. If you followed docker compose example, it's `mysql-chevereto`
* Database Port
     - Your MySQL server port. If you followed docker compose example, it's `3306`
* Database Name
     - Your MySQL chevereto database name. If you followed docker compose example, it's `your_mysql_chevereto_dbname`
* Database User
     - Your MySQL chevereto database username. If you followed docker compose example, it's `your_mysql_chevereto_username`
* Database User password
     - Your MySQL chevereto database user password. if you followed docker compose example, it's `your_mysql_chevereto_user_password`




## License

The docker image is released under the [MIT license](LICENSE)

Please note that [Chevereto](cheveretourl) is a product of [Rodolfo Berrios](http://rodolfoberrios.com/), this project aims mainly to install web server with installer.php to use free or paid version with docker.


## Usage

To create the image `mdelapenya/lamp`, execute the following command on the mdelapenya-docker-lamp folder:

```shell
docker build -t mdelapenya/lamp .
```

You can now push your new image to the registry:

```shell
docker push mdelapenya/lamp
```

## Running your LAMP docker image

Start your image binding the external ports 80 and 3306 in all interfaces to your container:

```shell
docker run -d -p 80:80 -p 3306:3306 mdelapenya/lamp
```

Test your deployment:

```shell
curl http://localhost/
```

Hello world!

## Loading your custom PHP application

In order to replace the "Hello World" application that comes bundled with this docker image,
create a new `Dockerfile` in an empty folder with the following contents:

```shell
FROM mdelapenya/lamp:latest
RUN rm -fr /app && git clone https://github.com/username/customapp.git /app
EXPOSE 80 3306
CMD ["/run.sh"]
```

replacing `https://github.com/username/customapp.git` with your application's GIT repository.
After that, build the new `Dockerfile`:

```shell
docker build -t username/my-lamp-app .
```

And test it:

```shell
docker run -d -p 80:80 -p 3306:3306 username/my-lamp-app
```

Test your deployment:

```shell
curl http://localhost/
```

That's it!

## Connecting to the bundled MySQL server from within the container

The bundled MySQL server has a `root` user with no password for local connections.
Simply connect from your PHP code with this user:

```php
<?php
$mysql = new mysqli("localhost", "root");
echo "MySQL Server info: ".$mysql->host_info;
?>
```

## Connecting to the bundled MySQL server from outside the container

The first time that you run the container, a new user `admin` with all privileges
will be created in MySQL with a random password. To get the password, check the logs
of the container by running:

```shell
docker logs $CONTAINER_ID
```

You will see an output like the following:

```shell
========================================================================
You can now connect to this MySQL Server using:

    mysql -uadmin -p47nnf4FweaKu -h<host> -P<port>

Please remember to change the above password as soon as possible!
MySQL user 'root' has no password but only allows local connections
========================================================================
```

In this case, `47nnf4FweaKu` is the password allocated to the `admin` user.

You can then connect to MySQL:

```shell
mysql -uadmin -p47nnf4FweaKu
```

Remember that the `root` user does not allow connections from outside the container -
you should use this `admin` user instead!

## Setting a specific password for the MySQL server admin account

If you want to use a preset password instead of a random generated one, you can
set the environment variable `MYSQL_PASS` to your specific password when running the container:

```shell
docker run -d -p 80:80 -p 3306:3306 -e MYSQL_PASS="mypass" mdelapenya/liferay-portal-mysql
```

You can now test your new admin password:

```shell
mysql -uadmin -p"mypass"
```

## Disabling .htaccess

`.htaccess` is enabled by default. To disable `.htaccess`, you can remove the following contents from `Dockerfile`

```shell
# config to enable .htaccess
ADD apache_default /etc/apache2/sites-available/000-default.conf
RUN a2enmod rewrite
```
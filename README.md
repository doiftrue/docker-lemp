Installation
============

1. Install Docker
2. Clone this repository
3. Perform the following commands:
   ```
   touch php/config.php.ini
   cp .env.sample .env
   make create-cert example.loc
   make d.up
   ```
4. Add `127.0.0.1  example.loc` to the `/etc/hosts` system file.
   On Windows, this file is located at: `C:\Windows\System32\Drivers\etc\hosts`.
5. Done!
6. Go to `http://example.loc` in your browser, and the `./sites/example.loc/www/index.php` file will handle this request.


How to add a new site
---------------------
In the instructions below, we assume your new site domain is: `mysite.loc`.

### Option 1:
1. Add `mysite.loc` to the system `hosts` file: `127.0.0.1  mysite.loc`.
2. Create the site folder and the main file in it: `./sites/mysite.loc/www/index.php`.
3. Done! Go to `http://mysite.loc`.

### Option 2 (configure all):
This option allows you to configure all aspects: nginx, site file structure, etc.
You need to use one of the examples from `nginx/sites-enabled-examples/`:

1. Add `mysite.loc` to the system `hosts` file: `127.0.0.1  mysite.loc`.
2. Copy the `example.conf` file to the `nginx/sites-enabled` folder.
3. Rename `example.conf` to `mysite.loc.conf` (this is not necessary, but it's more convenient for you).
4. Restart Docker containers: `$ make d.down && make d.up` to apply the new configuration in the nginx container.
5. Done! Go to `http://mysite.loc`.

### Option 3 (https, SSL)
To create an SSL-based site, follow the instructions from "Option 2" but use the `example-ssl.conf` file instead. Additionally, you need to create a self-signed certificate for your new site. Run the command:
```sh
make create-cert mysite.loc
```
If you are adding the site for the first time, you need to add the root certificate (ROOT CA) to your system and browser. See the instructions [here](certs/README.md).



PHP
---
Go into the container:

    make goto.php


xDebug
------
- See all parameters: https://xdebug.org/docs/all_settings
- See: [Upgrading from Xdebug 2 to 3](https://xdebug.org/docs/upgrade_guide)
- See: https://xdebug.org/docs/all_settings#xdebug.start_with_request

To enable xDebug, add the following configuration to the `php/config.php.ini` file:
```ini
[xdebug]
xdebug.mode = debug
xdebug.start_with_request = default
xdebug.client_host = host.docker.internal
xdebug.client_port = 9003
xdebug.idekey = PHPSTORM
```

To view current values, run:
```shell
php -i | grep xdebug
```

Or call the PHP function:
```php
xdebug_info(); exit;
```

To trigger xDebug under WP-CLI, use:
```shell
XDEBUG_TRIGGER=1 wp plugin list
```

### xDebug profiling

In `config.php.ini`:
```
xdebug.mode = profile
xdebug.output_dir = /var/app/_tmp_docker_lamp_xdebug_profiles
xdebug.profiler_output_name = cachegrind.%R.%u
;xdebug.trigger_value = ""
```

Viewer program: KCacheGrind. Install for Ubuntu:
```shell
sudo apt install kcachegrind
sudo apt install graphviz
```
Check that graphviz is installed correctly:
```
dot -V
```



Redis
-----
To use Redis correctly, you need to use the "redis" host. 
For example, if you use the WP plugin [Redis Object Cache](https://wordpress.org/plugins/redis-cache/), add the following constant to the `wp-config.php` file:

   define('WP_REDIS_HOST', 'redis');


DB
--
To create your site database, go to phpMyAdmin and create a database:

    $ make phpmyadmin

Go to: http://localhost:9200

OR you can do it using the MySQL console by entering the MySQL container:

    $ make goto.mysql
    # mysql -uroot -proot
    mysql> SHOW DATABASES;
    mysql> CREATE DATABASE test_database;


Nginx
-----
Go into the container:

    make goto.nginx
    // OR
    docker exec -it nginx sh

Use one of the following commands:

    nginx -t           # Test config
    nginx -T           # Test & Dump config
    nginx -s reload    # Reload the configuration file
    nginx -s quit      # Graceful shutdown

NOTE: Check the `fastcgi_params` inside the container at `/etc/nginx/fastcgi_params`.


Mail testing
------------
To test mailing on the site, "Mailpit" is used. As it is needed on demand, it wasn't added to the main docker-compose file. You need to run it (its container) separately using the following make command:

	$ make mailpit

Then go to <http://localhost:8025> 

**Additional info:**

> "Mailpit" is a lightweight, self-hosted email testing tool that simulates an SMTP server and provides a web interface for viewing and testing email deliveries.

> "msmtp" is a lightweight SMTP client used to send emails from the command line or scripts. It acts as an SMTP relay, forwarding emails to a specified SMTP server. It's often used in Unix-based systems as a simpler alternative to more complex mail transfer agents (MTAs) like sendmail or postfix.

The PHP container is configured to use the "msmtp" client to send emails. So, all emails sent from PHP will be caught by "Mailpit" by default. For this purpose, the "msmtp" and "msmtp-mta" packages are added to the PHP image (Dockerfile).

> "msmtp-mta" is a drop-in replacement for traditional Mail Transfer Agents (MTAs) like "sendmail" or "postfix". This allows programs that expect a sendmail binary to send emails using "msmtp" without changing the configuration, for example, PHP's mail() function.

"msmtp" is configured to use "Mailpit" as the SMTP server.

You can change the "msmtp" configuration in the `./php/msmtprc.ini` file. After any changes, you need to recreate the PHP container by running `$ make d.recreate`.

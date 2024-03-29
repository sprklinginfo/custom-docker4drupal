# Basic docker compose template for Drupal 80:80
This is a basic docker compose template to set up Drupal 8 local development environment

## For a new Drupal installation
### Step 1: 
Create a <proeject-directory>.
Copy the the everything from *custom-drupal8-docker*.

### Step 2:
Update .env file for environment variables. 

### Step 3:
Use composer to download the drupal codebase.
Remove *.keep* file from *drupal* folder first. 
```
$ cd <project-directory>
$ composer create-project drupal-composer/drupal-project:8.x-dev drupal ---stability dev --no-interaction
```

### Step 4:
```
$ docker-compose config
$ docker-compose up -d
```

### Step 5:
- Visit drupal frontend: http://drupal.test:8000
- Visit phpmyadmin page: http://pma.drupal.test:8000
- Visit portainer page: http://portainer.drupal.test:8000

## For an existing Drupal project
Follow the same instruction on Step 1 and Step 2 above.

### Step 3:
Copy the existing drupal code into drupal/web directory.
Update *settings.php* file in the following section:
```
$databases['default']['default'] = array (
  'database' => getenv('DB_NAME'),
  'username' => getenv('DB_USER'),
  'password' => getenv('DB_PASSWORD'),
  'prefix' => '',
  'host' => getenv('DB_HOST'),
  'port' => '3306',
  'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
  'driver' => getenv('DB_DRIVER'),
);
```
For local development, 'trusted_host_patterns' section may need to be updated as well:
```
$settings['trusted_host_patterns'] = [
   '^localhost',
   '^.+\.example\.com$',
   '^.+\.jsdelivr\.net$',
   '^.+\.test$',
 ];
 ```
Follow instruction on Step 4 and Step 5 above.

##To run drush command we can do it in the container:
```
$ winpty docker-compose exec php sh
```

##To install new drupal modules
In project folder of the host machine:
```
$ cd drupal/
$ composer require drupal/module-name
```

##To update Drupal Core and its dependencies
In project folder of the host machine:
```
$ cd drupal/
$ composer update drupal/core webflo/drupal-core-require-dev "symfony/*" --with-dependencies
```
## Add authentication to portainer container
Gernate the password for admin first:
```
$ docker run --rm httpd:2.4-alpine htpasswd -nbB admin pw4admin | cut -d ":" -f 2
```
The output is $2y$05$erBb3xrKAaI3GdSUobH2wOr3Kv8fzlPfz7CsB4OolwBZoXUL4I9xG
each $ in above string need to be escaped as $$
In portainer section change 'command' link like this:
```
command: --admin-password "$$2y$$05$$erBb3xrKAaI3GdSUobH2wOr3Kv8fzlPfz7CsB4OolwBZoXUL4I9xG" -H unix:///var/run/docker.sock
```
Note: we can also get md5 string on http://www.htaccesstools.com/htpasswd-generator/ site. 
## Access a container as root
```
winpty docker exec -u 0 -it my_drupal8_project_php bash
```
## How to enable xdebug
1. enable the following lines in PHP container, then recreate the container.
```
      PHP_XDEBUG: 1
      PHP_XDEBUG_DEFAULT_ENABLE: 1
      PHP_XDEBUG_REMOTE_CONNECT_BACK: 0
      PHP_IDE_CONFIG: serverName=my-ide
      PHP_XDEBUG_REMOTE_HOST: host.docker.internal # Docker 18.03+ Mac/Win
      PHP_XDEBUG_REMOTE_LOG: /tmp/php-xdebug.log
      PHP_XDEBUG_REMOTE_AUTOSTART: 1
```
2. Update hosts file with two more lines:
```
0.0.0.0     localhost
10.0.75.1   localhost
```
3. in PHPSTORM IDE
+ Open *Run > Edit Configurations* from the main menu, choose *Defaults > PHP Web Page* in the left sidebar
+ Click to *[...]* to the right of Server and add a new server
+ Enter name **my-ide** (as specified in PHP_IDE_CONFIG)
+ Enter any host, it does not matter
+ Check Use path mappings, select path to your project and enter /var/www/html in the right column (Absolute path on the server). Local *web/* -> */var/www/html/web* in the container.
+ Choose newly created server in "Server" for PHP Web Page
+ Save settings

4. In PHPSTORM IDE
Select menu *Run > Debug the newly created configuration*.

## Global Traefik container
Use *traefik-proxy2* folder as a global container

This template is adpated from Docker4Drupal:
  - [Docs](https://wodby.com/docs/stacks/drupal/local/#usage)
  - [Git Repository](https://github.com/wodby/docker4drupal)
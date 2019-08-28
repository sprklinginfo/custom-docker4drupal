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
  'database' => getenv('DATABASE_NAME'),
  'username' => getenv('DATABASE_USER'),
  'password' => getenv('DATABASE_PASSWORD'),
  'prefix' => '',
  'host' => getenv('DATABASE_HOST'),
  'port' => '3306',
  'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
  'driver' => 'mysql',
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


This template is adpated from Docker4Drupal:
  - [Docs](https://wodby.com/docs/stacks/drupal/local/#usage)
  - [Git Repository](https://github.com/wodby/docker4drupal)
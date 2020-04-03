# Laravel-Docker

* Create new laravel project

    ```bash
    composer create-project --prefer-dist laravel/laravel laravel-docker
    ```

* Create file `/docker/Dockerfile`

    ```dockerfile
    FROM php:7.3-apache-stretch
    ```

* Run:
    ```bash
    docker build -t laravel-docker -f docker/Dockerfile .
    ```
    * `-t laravel-docker` : tag name `-t`
    * `-f docker/Dockerfile` : file location `-f`
    * `.` : build context (current directory)

* Run: 
    
    ```bash
    docker run --rm -it laravel-docker /bin/bash
    ```
    * `--rm` remove container after running
    * `-it` interactive mode
    * Useful commands:
      * `php --version` : get php version
      * `apachectl -S` : debug your virtual host configuration
        ```bash
        VirtualHost configuration:
        *:80                   172.17.0.2 (/etc/apache2/sites-enabled/000-default.conf:1)
        ServerRoot: "/etc/apache2"
        Main DocumentRoot: "/var/www/html"
        Main ErrorLog: "/var/log/apache2/error.log"
        Mutex watchdog-callback: using_defaults
        Mutex default: dir="/var/run/apache2/" mechanism=default 
        Mutex mpm-accept: using_defaults
        PidFile: "/var/run/apache2/apache2.pid"
        Define: DUMP_VHOSTS
        Define: DUMP_RUN_CFG
        User: name="www-data" id=33
        Group: name="www-data" id=33
        ```
    
**Apache with a Dockerfile**

* [Docker documentation](https://hub.docker.com/_/php)
 
    ```dockerfile
    FROM php:7.2-apache
    COPY src/ /var/www/html/
    ```

**Info**

* To see how to copy files/folders between a container and the local filesystem :

`docker cp --help` 

* Let's try to copy a file from our container to local dir `/docker`

 - 1 - run the container in detach mode
    ```bash
    docker run --rm -d laravel-docker
    ```
 - 2 - list running containers
    ```bash
    docker ps 

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    1ef089b57ad6        laravel-docker      "docker-php-entrypoi…"   6 seconds ago       Up 2 seconds        80/tcp              happy_cray
    ```
 - 3 - SSH into container
    ```bash
    docker exec -it 1e bash
   ```
 - 4 - Get file path we want to copy 
   ```bash
   apachectl -S
   ```
 
      filepath: `/etc/apache2/sites-enabled/000-default.conf` and it's a symlink
      Get real filepath:
      ```bash
        root@1ef089b57ad6:/var/www/html# ls -la /etc/apache2/sites-enabled/000-default.conf
        lrwxrwxrwx 1 root root 35 Mar 31 19:09 /etc/apache2/sites-enabled/000-default.conf -> ../sites-available/000-default.conf
      ```
 - 5 - Copy the file:
     ```bash
     docker cp 1e:/etc/apache2/sites-available/000-default.conf docker/vhost.conf
     ```
     * Important: `1e` is the CONTAINER_ID (`docker ps`)

* **Run our application**:
  * Setup our config:
    ```dockerfile
    FROM php:7.3-apache-stretch
    
    COPY . /var/www/html
    
    COPY /docker/vhost.conf /etc/apache2/sites-available/000-default.conf
    ```
    
      * IMPORTANT: Build may takes few minutes because of `RUN chown -R www-data:www-data /var/www/html`
  
  * run command
      ```bash
        docker build -t laravel-docker -f docker/Dockerfile .
      
        Sending build context to Docker daemon  46.19MB
        Step 1/3 : FROM php:7.3-apache-stretch
         ---> 2cef340dd582
        Step 2/3 : COPY . /var/www/html
         ---> c105e8508c01
        Step 3/3 : COPY /docker/vhost.conf /etc/apache2/sites-available/000-default.conf
         ---> 24df67826119
        Successfully built 24df67826119
        Successfully tagged laravel-docker:latest
      ```
  
  * run our container:
      ```bash
      docker run --rm -p 8080:80 laravel-docker
      ```

  * `http://localhost:8080` Error! 
  
    ```bash
    UnexpectedValueException
    The stream or file "/var/www/html/storage/logs/laravel.log" could not be opened: failed to open stream: Permission denied
    http://localhost:8080/
    ```

* **Enable mod_rewrite** 
  
  * SSH into our container:
    ```bash
    docker ps
    
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
    8184a8219ba8        laravel-docker      "docker-php-entrypoi…"   4 seconds ago       Up 1 second         0.0.0.0:8080->80/tcp   stupefied_mclean
    
    docker exec -it 81 bash
    ```
  
  * List apache modules
    ```bash
    a2enmod 
    Your choices are: access_compat actions alias allowmethods asis auth_basic auth_digest auth_form authn_anon authn_core authn_dbd authn_dbm authn_file authn_socache authnz_fcgi authnz_ldap authz_core authz_dbd authz_dbm authz_groupfile authz_host authz_owner authz_user autoindex buffer cache cache_disk cache_socache cern_meta cgi cgid charset_lite data dav dav_fs dav_lock dbd deflate dialup dir dump_io echo env expires ext_filter file_cache filter headers heartbeat heartmonitor http2 ident imagemap include info lbmethod_bybusyness lbmethod_byrequests lbmethod_bytraffic lbmethod_heartbeat ldap log_debug log_forensic lua macro mime mime_magic mpm_event mpm_prefork mpm_worker negotiation php7 proxy proxy_ajp proxy_balancer proxy_connect proxy_express proxy_fcgi proxy_fdpass proxy_ftp proxy_hcheck proxy_html proxy_http proxy_http2 proxy_scgi proxy_wstunnel ratelimit reflector remoteip reqtimeout request rewrite sed session session_cookie session_crypto session_dbd setenvif slotmem_plain slotmem_shm socache_dbm socache_memcache socache_shmcb speling ssl status substitute suexec unique_id userdir usertrack vhost_alias xml2enc
    Which module(s) do you want to enable (wildcards ok)?
    ```

  * Enable `rewrite` module
    ```bash
    $ a2enmod rewrite
    Enabling module rewrite.
    To activate the new configuration, you need to run:
      service apache2 restart
    ```

  * Add `a2enmod rewrite` to our '/docker/Dockerfile'

  * We can disable `.htaccess` and move its content to `vhost` configuration in `docker/Dockerfile/vhost`

## Switch to docker-compose:

* Now we will translate command `docker run --rm -p 8080:80 laravel-docker` to `docker-compose` file!

* Create `docker-compose.yaml`

    ```yaml
    version: "3"
    services:
      app:
        image: laravel-www
        container_name: laravel-www
        build:
          context: .
          dockerfile: docker/Dockerfile
        ports:
          - 8080:80
    ```
    
    * After creating the file rebuild and run the container in detached mode:
        ```bash
        docker-compose up --build -d
        ```

    * Now we can SSH into container using the service name `app`
        ```bash
        docker-compose exec app /bin/bash
        ```

* **FIX AN ISSUE**
  * [Dockerfile - Speed Up The Setting of Permissions](https://blog.programster.org/dockerfile-speed-up-the-setting-of-permissions)
    The `COPY` docker command supports `--chown` which removes the need to do this as a separate `RUN` step
    * Change this:
      ```dockerfile
      COPY . /var/www/html
      RUN chown -R www-data:www-data /var/www/html
      ```
    * To this: 
      ```dockerfile
      COPY --chown=www-data:www-data . /var/www/html
      ```

* Create `.dockerignore` file

  ```
  /.idea
  /.git
  /node_modules
  ```

    * Before the docker CLI sends the context to the docker daemon, it looks for a file named `.dockerignore` in the root directory of the context. If this file exists, the CLI modifies the context to exclude files and directories that match patterns in it

### Add MYSQL Service

```yaml
version: "3"
services:
  app:
    image: laravel-www
    container_name: laravel-www
    build:
      context: .
      dockerfile: docker/Dockerfile
    ports:
      - 8080:80
  mysql:
    container_name: laravel-mysql
    image: mysql:5.7
    environment:
      MYSQL_DATABASE: homestead
      MYSQL_ROOT_PASSWORD: root
      MYSQL_USER: homestead
      MYSQL_PASSWORD: secret
    ports:
      - 13306:3306
```

* Install pdo_extension
  * file `/docker/Dockerfile`
    ```dockerfile
    RUN docker-php-ext-install pdo_mysql
    ```
  * TO check that it's installed, SSH into our container and run:
    ```bash
    php -m | grep mysql  
    ```

* Build app, SSH into container and run migration!
    ```bash
    docker-compose up --build -d
    docker-compose exec app bash
    php artisan migrate
    ```

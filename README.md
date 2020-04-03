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


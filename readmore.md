* **Volumes**

  * `docker volume inspect`	Display detailed information on one or more volumes
  * `docker volume ls`	List volumes
  * `docker volume prune`	Remove all unused local volumes
  * `docker volume rm	Remove` one or more volumes

* **Using Docker Compose**

    `$ docker-compose up --build`

    * Or, if you want to run it in the background

    `$ docker-compose up -d --build`

    * Now list all the containers running

    `$ docker-compose ps`

* **Stopping Containers with Docker Compose**
    * From the root of the project

    `$ docker-compose stop`

* **Additional Docker Compose Commands**
    * List running containers that Docker Compose is managing

    `$ docker-compose ps`

    * Restart the containers

    `$ docker-compose restart`

    * Restart a specific container
    * matches the service key in `docker-compose.yml`

    `$ docker-compose restart phpinfo`

    * Remove stopped containers

    `$ docker-compose stop && docker-compose rm`

    * Stop containers and remove containers, networks,
    * volumes, and images created

    `$ docker-compose down`

    * Remove named volumes

    `$ docker-compose down --volumes`

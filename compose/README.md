# Working with Related Containers

Another usage pattern for Docker is to package not only the application, but also its dependencies, in to Docker containers. In this section you'll run the Liberty MongoDB sample in one container with MongoDB running in another and then use Docker Compose to manage them both.

## Resolving dependencies via linking
You'll begin by using Docker linking to allow the Liberty application to resolve the MongoDB instance.

1. Create an image with WebSphere Liberty and the MongoDB feature as follows.

 1. Switch to the `compose` directory containing this README by typing:

    ```bash
    $ cd compose
    ```
 2. View the contents of the Dockerfile by typing:

    ```bash
    $ cat Dockerfile
    ```
The file adds the Liberty mongodb-2.0 feature from the Liberty online repository using the `installUtility` command and then copies in the server configuration.

2. View the server.xml by running:

    ```bash
    $ cat mongoDBSample/server.xml
    ```
The important part to note is the <mongo> stanza that specifies that the hostname should be `db`.

3. Build the application with the command:

    ```bash
    $ docker build -t app .
    ```
4. Pull down MongoDB from Docker Hub to your local cache using the command:

    ```bash
    $ docker pull mongo
    ```
5. Now start the MongoDB server using the following command:

    ```bash
    $ docker run -d --name mongodb mongo
    ```
6. Now start an instance of the app container using the following command:

    ```bash
    $ docker run -d --link mongodb:db -p 80:9080 --name app app
    ```
The `--link` option indicates that the `mongodb` container should be made available to this container under the alias `db`.

7. Use the following command to wait for the server to start:

    ```bash
    $ docker logs --tail=all -f app
    ```

8. Use this docker command to check the IP address the server is on, similar to the "Adding an Application" Tutorial:

    ```bash
    $ docker ps
    ```

9. View the running application by opening a web browser at this IP address followed by `/mongoDBApp`. Each time the page is refreshed it writes more users in to the MongoDB base.

10. Clean up the containers and images with the following commands:

    ```bash
    $ docker kill $(docker ps -q)
    $ docker rm $(docker ps -aq)
    $ docker rmi app
    ```

Note: By default, container linking only works between containers on a single host. Across multiple hosts there are a number of options:
  * Pass the address of the remote container as an environment variable to the container. The Liberty ``$(env.XXX)` syntax makes it easy to substitute variables in to server.xml.
  * Use some form of service registry to locate dependencies (e.g. etcd or consul).
  * As of Docker 1.10, use a Docker overlay network to create a software defined network spanning the multiple hosts.

## Docker Compose for container orchestration
Docker Compose provides a simple file format to define the relationship between containers that make up a single application.

1. View the `docker-compose.yml` in the current directory that defines the application using:

    ```bash
    $ cat docker-compose.yml
    ```
Note that it defines two services: web and db and links them together.

2. Start the application with the following command (as with the `docker` command, the `-d` option starts the containers in the background):

    ```bash
    $ docker-compose up -d
    ```
3. The following command will then show an aggregated view of the logs from the two services:

    ```bash
    $ docker-compose logs
    ```
4. `Ctrl-C` to exit the logs view.

5. You can scale up the number of containers for the web service using the command:

    ```bash
    $ docker-compose scale web=2
    ```
6. Use the following command to list all of the containers that make up the application:

    ```bash
    $ docker-compose ps
    ```
7. Execute the following commands to demonstrate that the two Liberty containers are using the same MongoDB instance:

    ```bash
    $ curl $(docker-compose port --index=1 web 9080)/mongoDBApp
    $ curl $(docker-compose port --index=2 web 9080)/mongoDBApp
    ```
7. (b) You should see that the two application instances are serving data from the same MongoDB database with the second command returning the users created by the first command plus ten more.

    ```bash
    $ docker-compose ps
    ```

8. Clean up the containers with the following commands:

    ```bash
    $ docker-compose kill
    $ docker-compose rm -f
    ```

Note: Docker Compose can only communicate with a single Docker API endpoint and therefore, by default, is restricted to creating containers on a single node. However, when used in conjunction with Docker Swarm, which exposes a single API endpoint for managing a cluster of Docker nodes, it is able to schedule servers over multiple nodes.

## Congratulations

Congratulations on completing this tutorial. You may now like to look at the tutorial on [Deploying to IBM Containers on Bluemix](../bluemix).

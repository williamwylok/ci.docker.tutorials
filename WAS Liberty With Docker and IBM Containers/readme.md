WAS Liberty With Docker and IBM Containers
==========================================

February 2016 edition

# Introduction
Although Docker is being ported to other platforms, in its original form it runs natively on Linux using capabilities provided by the Linux kernel. When using Docker in a Windows or OS/X development environment it is typical to install the Docker Toolbox (https://www.docker.com/docker-toolbox) which provides the Docker command line running natively, talking to the Docker engine running in a lightweight Linux Virtual Box virtual machine.

## Prerequisites:
1. [Docker Toolbox](https://www.docker.com/products/docker-toolbox/) (1.10 or later) for Windows or Mac
2. [Cloud Foundry CLI](http://docs.cloudfoundry.org/cf-cli/install-go-cli.html) and the [IBM Containers plug-in](https://console.ng.bluemix.net/docs/containers/container_cli_cfic.html)
3. Download this repository onto your local machine

To complete the final section of this lab you will also need an IBM ID registered with IBM Bluemix. If you do not have an IBM ID, or have not signed up for Bluemix, and wish to complete this section of the lab then sign up for a free 30-day trial at https://console.ng.bluemix.net/registration/.

## Running a Liberty container
In this section you'll run your first Liberty server under Docker using the websphere-liberty image that is available from the online Docker Hub repository of images.

1. Open the Docker Quickstart Terminal on Windows or a Mac terminal.
2. Populate the local image cache with WebSphere Liberty by using the following command to pull it down from Docker Hub:

    ```bash
    $ docker pull websphere-liberty
    ```
This will pull down the layers that make up the image. The total image size, including the base Ubuntu image on which it builds, is around 570MB.
3. Execute the following command to see the layers that make up the Liberty image:

    ```bash
    $ docker history websphere-liberty
    ```
The layers represent the commands that were issued when the image was built (specify the –-no-trunc option if you want to see the full commands). The bottom four layers represent the `ubuntu` image. After setting the maintainer and installing wget, the Dockerfile downloads and installs an IBM JRE and the WebSphere Liberty kernel. It then creates a server and then sets the metadata around the image such as the ports to expose and the command to run. It then adds additional features using the installUtility command to build up to the full set of Java EE7 features.

4. You can see the total size of the image by running the command:

    ```bash
    $ docker images websphere-liberty
    ```
5. Run the image with the following command (the `-i` and `-t` options specify that an interactive terminal is required):

    ```bash
    $ docker run –i -t websphere-liberty
    ```
6. The server isn't much use though as we didn’t expose any ports to access it. Type `Ctrl-C` to stop the server. (If you failed to specify the `-i –t` options then you will have to kill the process from another terminal.)

7. This time map the port 9080 from the container to port 80 on the host VM:

    ```bash
    $ docker run -d -p 80:9080 --name wlp websphere-liberty
    ```
Note that this time you have specified the `-d` option to run the container in the background, and we have given it a name so that we can subsequently refer to it by this name rather than using the generated name or id.
8. Use the following command to follow the logs of the container as it starts up:

    ```bash
    $ docker logs --tail=all -f wlp 
    ```
Once the server is ready to run, type `Ctrl-C` to return to the command prompt.
9. Open a browser at `localhost` to view the Liberty welcome page.
10. Return to the terminal and enter the following command to see the processes running in the container:

    ```bash 
    $ docker top wlp
    ```
You will see the `server` script and the child Java process.
11. If you execute the following commands, you can see that these processes can also be seen at the level of the Docker host. They are normal Linux processes whose visibility of the rest of the host are constrained by the use of Linux namespaces.

    ```bash
    $ docker-machine ssh
    $ ps aux | grep wlp
    $ exit
    ```
12. Docker permits us to run other processes inside the namespaces of an existing container. This is primarily used for diagnostic purposes.
  1. For example, type the following command to start an interactive session inside the running container:

    ```bash
    $ docker exec -it wlp bash
    ```
You will see the command prompt change indicating that you are now running as the root user within the container.
  2. Execute the following command to show all of the processes running within the container namespace.

    ```bash
    $ ps aux
    ```
Note that within the container you do not have visibility of other processes running on the host.
  3. Execute the following command inside the container to view the contents of the server log file:

    ```bash
    $ cat /logs/messages.log
    ```
  4. Enter the following command to end the terminal session. The container will continue to run.

    ```bash
    $ exit
    ```
13. It is also possible to copy files out of the container.
 1. For example type the following command to copy the messages.log file from the container `wlp` to the current directory:

    ```bash
    $ docker cp wlp:/logs/messages.log .
    ```
 2. Then run execute the following command to view the file:

    ```bash
    $ cat messages.log
    ```
14. Finally, run the following commands to clear up the container:

    ```bash
    $ docker stop wlp
    $ docker rm wlp
    ``` 

## Handling multiple containers
Explicitly setting ports and names is fine if you only have a single container but what if you are trying to run multiple instances of an image?

1. Type the following command to run the Liberty image again:

    ```bash
    $ C=$(docker run -d -P websphere-liberty)
    ```
This time you have specified the `-P` flag to indicate that Docker should automatically select ports on the host to map those exposed by the image to. You have also saved the identifier for the new container in the variable `$C`.
2. Run the following command to see the ports on the host that Docker has mapped 9080 and 9443 to:

    ```bash
    $ docker ps
    ```
3. The `docker port` command can be used to retrieve the mapping for a specific port. For example, you can retrieve the Liberty welcome page for this container by typing the command:

    ```bash
    $ curl $(docker port $C 9080)
    ```
4. Using `docker inspect $C` reveals lots more interesting information about the running container. The command accepts filters in the golang template format so, for example, it is possible to retrieve just the IP address for the container using:

    ```bash
    $ docker inspect --format '{{ .NetworkSettings.IPAddress }}' $C
    ```
5. You can now run another instance of the container using the same command and Docker will allocate different ports:

    ```bash
    $ D=$(docker run -d -P websphere-liberty)
    ```
6. Execute the following command to confirm that you now have two containers running:

    ```bash
    $ docker ps
    ```
7. Run the following command to see a live stream of stats from the running containers:

    ```bash
    $ docker stats $C $D
    ```
Note that, having not expressed any constraints, each container has the full memory allocated to virtual machine available to it.
8. Enter `Ctrl-C` to terminate the stats stream.
9. Clean up the containers with the following commands:

    ```bash
    $ docker kill $C $D
    $ docker rm $(docker ps -aq)
    ```

## Adding an application
You now know how to run one or more Liberty servers under Docker – next you need to add an application to the mix. There are a number of different approaches to running an application on the Liberty Docker Hub image which you’ll explore in this section of the lab.

### Mounting an application on the image
The first method is simply to mount an application or directory from the host when the container is started.

1. Change down in to the `app` directory of the lab materials:

    ```bash
    $ cd app
    ```
2. Run the following command to copy the sample Servlet App to your docker host VM:

    ```bash
    $ docker-machine scp ServletApp.war $(docker-machine active):/tmp/ServletApp.war
    ```
3. Use the following command to run a container with WebSphere Liberty:

    ```bash
    $ docker run -d -p 80:9080 --name=app -v /tmp/ServletApp.war:/config/dropins/app.war websphere-liberty
    ```
The important part of the command is the `-v` option which indicates that the ServletApp.war from the current directory (Docker requires an absolute path) should be mounted in to the dropins directory of the Liberty configuration.
4. Use the following command to watch the server start:

    ```bash
    $ docker logs --tail=all -f app
    ```
You should see log entries indicating that the application has been started.
5. View it in a browser at `http://localhost/app`.
6. Clean up the server with the following commands:

    ```bash
    $ docker kill app
    $ docker rm app
    ``` 

### Creating an application layer
The approach above is fine for development purposes allowing you to rapidly update the applica-tion but mounting a file from the host breaks the portability that is often a driver to use Docker in the first place. The next approach is to build an image with the application contained within a lay-er.

1. In the current directory, list the contents of ‘Dockerfile’ with the following command:

    ```bash
    $ cat Dockerfile
    ```
It simply starts from the websphere-liberty image and adds the ServletApp.war from the current directory in to the dropins directory.
2. Build an image tagged `app` using the current directory as context using the command:

    ```bash
    $ docker build -t app .
    ```
3. To run the new built image, use the command:

    ```bash
    $ docker run -d -p 80:9080 --name=app app
    ```
4. Repeat steps 3-5 from the previous section to watch the server start, verify that it is running, and then clean up.

### Using a data volume container
The approach from the previous section means that you have neatly packaged a single image that contains everything needed to run the application. It does, however, mean that if we want to move up to a new version of the Liberty image we have to rebuild our application as well. The fol-lowing approach using a data volume container to separate the application and the server runtime in two separate images.

1. Run the image that you created in the previous section with the following command:

    ```bash
    $ docker run -v /config --name=app app true 
    ```
This time, the `-v` flag is being use to export a volume from the server. The last parameter of `true` is the command that the container will run. This will exit immediately leaving you with a stopped container that exposes the server configuration containing the application.
Note: You have just reused the existing image for convenience here. You are not going to use the Liberty server runtime in it and could equally have used an image created from `ubuntu`.
2. Now run the following command to start a Liberty container:

    ```bash
    $ docker run --volumes-from app -d -p 80:9080 websphere-liberty
    ```
The key option this time is the `--volumes-from` flag which indicates that the volumes exported by the container `app` should be mounted on this container.
3. Once again, use the following command to watch the server start (where the `docker ps -lq` command returns the ID of the last container run):

    ```bash
    $ docker logs --tail=all -f $(docker ps –lq)
    ```
You should see log entries indicating that the application has been started.
4. View the web app in the browser at `http://localhost/app`.
5. Clean up the server and the two containers with the following commands:

    ```bash
    $ docker kill $(docker ps -lq)
    $ docker rm $(docker ps -aq)
    ```
6. Remove the application image with the command:

    ```bash
    $ docker rmi app
    ``` 

## Running dependencies in containers
Another usage pattern for Docker is to package not only the application, but also its dependencies in to Docker containers. In this section you'll run the Liberty MongoDB sample in one container with MongoDB running in another and then use Docker Compose to manage them both.

### Resolving dependencies via linking
You'll begin by using Docker linking to allow the Liberty application to resolve the MongoDB in-stance.

1. Create an image with WebSphere Liberty and the MongoDB feature as follows.
 1. Switch to the `mongo` directory by typing:

    ```bash
    $ cd ../ mongo
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
4. Pull down MongoDB from DockerHub to your local cache using the command:

    ```bash
    $ docker pull mongodb
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
8. Browse to `http://localhost/mongoDBApp`. Each time the page is refreshed it writes more users in to the MongoDB base.
9. Clean up the containers and images with the following commands:

    ```bash
    $ docker kill $(docker ps -aq)
    $ docker rm $(docker ps -aq)
    $ docker rmi app
    ```

Note: Container linking only works between containers on a single host. Across multiple hosts there are a number of options:
  * Pass the address of the remote container as an environment variable to the container. The Liberty ‘$(env.XXX)’ syntax makes it easy to substitute variables in to server.xml.
  * Use some form of service registry to locate dependencies (e.g. etcd, consul, …).
  * Use a Docker overlay network to create a software defined network spanning the multiple hosts.

### Docker Compose for container orchestration
Docker Compose provides a simple file format to define the relationship between containers that make up a single application.

1. View the docker-compose.yml in the current directory that defines the application using:

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
You should see that the two web app instances are serving data from the same MongoDB database with the second command returning the users created by the first command plus ten more.

    ```bash
    $ docker-compose ps
    ```
8. Clean up the containers with the following commands:

    ```bash
    $ docker-compose kill
    $ docker-compose rm -f
    ```
    
Note: Docker Compose can only communicate with a single Docker API endpoint and therefore, by default, is restricted to creating containers on a single node. However, when used in conjunction with Docker Swarm, which exposes a single API endpoint for managing a cluster of Docker nodes, it becomes much more powerful.

## Deploying to IBM Containers
In this last section, you’ll deploy the same Liberty MongoDB sample to the IBM Containers service on IBM Bluemix. 

1. Log in IBM Bluemix using the Cloud Foundry CLI. You will be prompted to provide your IBM ID and password.

    ```bash
    $ cf login -a https://api.ng.bluemix.net
    ```
2. If you haven’t use IBM Containers before, you need to specify a Docker registry namespace to use with your account.

    ```bash
    $ cf ic namespace set <namespace>
    ```
3. Initialize the IBM Containers plugin:

    ```bash
    $ cf ic init
    ```
Note: When the init command is run you will be presented with two options. Option one allows you to simultaneously manage IBM Containers and your local Docker host through “cf ic” commands. Option two uses the Docker CLI, overriding the local Docker environment to connect to IBM Containers. This tutorial will use the first option.  
4. Copy the MongoDB Docker Hub image to your Bluemix registry.

    ```bash
    $ cf ic cpi mongo registry.ng.bluemix.net/<namespace>/mongo
    ```
5. Run an instance of the Mongo image that you just copied across called mongodb

    ```bash
    $ cf ic run -d --name mongodb registry.ng.bluemix.net/<namespace>/mongo
    ```
6. Change to the `bluemix` directory in the materials folder.

    ```bash
    $ cd ../bluemix/
    ```
7. You could re-use the existing 'app' image that you built earlier but it is quicker to rebuild the image in IBM Bluemix than upload it. Look at the Dockerfile using the following com-mand. Notice the base image being used is the ibmliberty image in the IBM Containers registry.

    ```bash
    $ cat Dockerfile
    ```
8. Build the app in IBM Bluemix:

    ```bash
    $ cf ic build -t registry.ng.bluemix.net/<namespace>/app .
    ```
9. Run the app that you just built linking the mongodb app as you did previously.

    ```bash
    $ cf ic run -d --link mongodb:db -p 80:9080 --name app registry.ng.bluemix.net/<namespace>/app
    ```
10. Check all the containers are up and running:

    ```bash
    $ cf ic ps
    ```
11. Access the application using the allocated IP address.

    ```bash
    $ curl $(cf ic port app 9080)/mongoDBApp
    ```
10. Clean up the containers and images with the following commands:

    ```bash
    $ cf ic kill $(cf ic ps -aq)
    $ cf ic rm $(cf ic ps -aq)
    $ cf ic rmi registry.ng.bluemix.net/<namespace>/app
    $ cf ic rmi registry.ng.bluemix.net/<namespace>/mongo
    ```

## Congratulations
Congratulations! You have completed this lab. You now know how to build a Docker image con-taining WebSphere Liberty and your application and run it locally or on the IBM Containers service on IBM Bluemix.

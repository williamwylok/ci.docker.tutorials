# Adding an Application
The tutorial [Running WebSphere Liberty Containers](../liberty) described how to run one or more Liberty servers under Docker. In this tutorial you will add an application to the mix. There are a number of different approaches to running an application on the Liberty Docker Hub image which you'll explore in this section of the lab.

## Mounting an application on the image
The first method is simply to mount an application or directory from the host when the container is started.

1. Change down in to the `app` directory containing the README for this tutorial:

    ```bash
    $ cd app
    ```
2. Run the following command to copy the sample Servlet App to your Docker host virtual machine:

    ```bash
    $ docker-machine scp ServletApp.war $(docker-machine active):/tmp/ServletApp.war
    ```
3. Use the following command to run a container with WebSphere Liberty:

    ```bash
    $ docker run -d -p 80:9080 --name=app -v /tmp/ServletApp.war:/config/dropins/app.war websphere-liberty
    ```
The important part of the command is the `-v` option which indicates that the ServletApp.war from the directory on the Docker host should be mounted in to the `dropins` directory of the Liberty server configuration.
4. Use the following command to watch the server start:

    ```bash
    $ docker logs --tail=all -f app
    ```
You should see log entries indicating that the application has been started.
5. Run the following Docker Machine command to identify the IP address allocated to the current active virtual machine:

    ```bash
    $ docker-machine ip $(docker-machine active)
    ```
6. View the running application by opening a web browser at this IP address followed by '/app'.
7. Clean up the server with the following commands:

    ```bash
    $ docker kill app
    $ docker rm app
    ```

### Creating an application layer
The approach above is fine for development purposes, allowing you to rapidly update the application but mounting a file from the host breaks the portability that is often a driver to use Docker in the first place. The next approach is to build an image with the application contained within a layer.

1. In the current directory, list the contents of `Dockerfile` with the following command:

    ```bash
    $ cat Dockerfile
    ```
It simply starts from the `websphere-liberty` image and adds the ServletApp.war from the current directory in to the `dropins` directory.
2. Build an image tagged `app` using the current directory as context using the command:

    ```bash
    $ docker build -t app .
    ```
3. To run the newly built image, use the command:

    ```bash
    $ docker run -d -p 80:9080 --name=app app
    ```
4. Repeat steps 4-7 from the previous section to watch the server start, verify that it is running, and then clean up.

### Using a data volume container
The approach from the previous section means that you have neatly packaged a single image that contains everything needed to run the application. It does, however, mean that if we want to move up to a new version of the Liberty image we have to rebuild our application as well. The following approach using a data volume container to separate the application and the server runtime in two separate images.

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
    $ docker logs --tail=all -f $(docker ps -lq)
    ```
You should see log entries indicating that the application has been started.
4. Run the following Docker Machine command to identify the IP address allocated to the current active virtual machine:

    ```bash
    $ docker-machine ip $(docker-machine active)
    ```
5. View the running application by opening a web browser at this IP address followed by '/app'
6. Clean up the server and the two containers with the following commands:

    ```bash
    $ docker kill $(docker ps -lq)
    $ docker rm $(docker ps -aq)
    ```
7. Remove the application image with the command:

    ```bash
    $ docker rmi app
    ```

## Congratulations

Congratulations on completing this tutorial. You may now like to look at the tutorial on [Working with Related Containers](../compose).

# Running WebSphere Liberty Containers

In this tutorial you'll run your first Liberty server under Docker using the `websphere-liberty` image that is available from the online Docker Hub repository.

## Running a single container

1. Open the Docker Quickstart Terminal on Windows or a Mac terminal.
2. Populate the local image cache with WebSphere Liberty by using the following command to pull it down from Docker Hub:

    ```bash
    $ docker pull websphere-liberty
    ```
   
   This will pull down the layers that make up the image. The total image size, including the base Ubuntu image on which it builds, is just over 500MB.
3. Execute the following command to see the layers that make up the Liberty image:

    ```bash
    $ docker history websphere-liberty
    ```
    The layers represent the commands that were issued when the image was built (specify the --no-trunc option if you want to see the full commands). The bottom layers represent the `ubuntu` image. After setting the maintainer and installing wget and unzip, the Dockerfile downloads and installs an IBM JRE and the WebSphere Liberty kernel. It then creates a server and then sets the metadata around the image such as the ports to expose and the command to run. It then adds additional features using the `installUtility` command to build up to the full set of Java EE7 features.

4. You can see the total size of the image by running the command:

    ```bash
    $ docker images websphere-liberty
    ```
5. Run the image with the following command (the `-i` and `-t` options specify that an interactive terminal is required):

    ```bash
    $ docker run -i -t websphere-liberty
    ```
6. The server isn't much use though as we didn't expose any ports to access it. Type `Ctrl-C` to stop the server. (If you failed to specify the `-i -t` options then you will have to kill the process from another terminal.)

7. This time map the port 9080 from the container to port 9080 on the host virtual machine:

    ```bash
    $ docker run -d -p 9080:9080 --name wlp websphere-liberty
    ```
   Note that this time you have specified the `-d` option to run the container in the background, and we have given it a name so that we can subsequently refer to it by this name rather than using the generated name or id.
8. Use the following command to follow the logs of the container as it starts up:

    ```bash
    $ docker logs -f wlp
    ```
    Once the server is ready to run, type `Ctrl-C` to return to the command prompt.
9. Open a web browser and enter the IP address of the machine running the container to view the Liberty welcome page.
    1. If you are using docker-machine on your desktop (Mac or Windows 7), then run docker-machine ip default to get the docker machine ip
    
    ```bash
    $ docker-machine ip default
    ```
    2. Open browser to 'http://docker-machine_ip:9080/'
    
10. Return to the terminal and enter the following command to see the processes running in the container:

    ```bash
    $ docker top wlp
    ```
    You will see the `server` script and the child Java process.
11. Docker permits us to run other processes inside the namespaces of an existing container. This is primarily used for diagnostic purposes.

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
12. It is also possible to copy files out of the container.

    1. For example type the following command to copy the messages.log file from the container `wlp` to the current directory:

        ```bash
        $ docker cp wlp:/logs/messages.log .
        ```
      
    2. Then run execute the following command to view the file:

        ```bash
        $ cat messages.log
        ```
        
13. Using `docker inspect wlp` reveals lots more interesting information about the running container. The command accepts filters in the golang template format so, for example, it is possible to retrieve just the IP address for the container using:

    ```bash
    $ docker inspect --format '{{ .NetworkSettings.IPAddress }}' wlp
    ```
    
14. Finally, run the following commands to clear up the container:

    ```bash
    $ docker stop wlp
    $ docker rm wlp

    ```

## Congratulations

Congratulations on completing this tutorial. You may now like to look at the tutorial on [Adding an Application](../app).

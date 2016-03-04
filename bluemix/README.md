# Deploying to IBM Containers

In this tutorial you will take the same sample application used in the [Working with Related Containers](../compose) tutorial and deploy it to the IBM Containers service running on IBM Bluemix.

To complete this tutorial you will need an IBM ID registered with IBM Bluemix. If you do not have an IBM ID, or have not signed up for Bluemix, and wish to complete this section of the lab then sign up for a [free 30-day trial](https://console.ng.bluemix.net/registration/). You will then need the [Cloud Foundry CLI](http://docs.cloudfoundry.org/cf-cli/install-go-cli.html) with the [IBM Containers plug-in](https://console.ng.bluemix.net/docs/containers/container_cli_cfic.html) installed.

1. Log in IBM Bluemix using the Cloud Foundry CLI. You will be prompted to provide your IBM ID and password.

    ```bash
    $ cf login -a https://api.ng.bluemix.net
    ```
2. If you haven't use IBM Containers before, you need to specify a unique registry namespace to use with your account. In the following command, substitute a value for <namespace> that must be a combination of lower-case letters and numbers, and must start with a letter.

    ```bash
    $ cf ic namespace set <namespace>
    ```
3. Initialize the IBM Containers plugin:

    ```bash
    $ cf ic init
    ```
Note: When the init command is run you will be presented with two options. Option one allows you to simultaneously manage IBM Containers and your local Docker host through 'cf ic' commands. Option two uses the Docker CLI, overriding the local Docker environment to connect to IBM Containers. This tutorial will use the first option.  
4. Copy the MongoDB Docker Hub image to your private registry, remembering to insert your namespace.

    ```bash
    $ cf ic cpi mongo registry.ng.bluemix.net/<namespace>/mongo
    ```
5. Run an instance of the Mongo image that you just copied across called mongodb

    ```bash
    $ cf ic run -d --name mongodb registry.ng.bluemix.net/<namespace>/mongo
    ```
6. Change to the `bluemix` directory containing this README.

    ```bash
    $ cd bluemix
    ```
7. Look at the Dockerfile that will be used to build the application image using the following command. Notice the base image being used is the `ibmliberty` image that is published in the IBM Containers registry.

    ```bash
    $ cat Dockerfile
    ```
8. Build the app in IBM Containers, remembering to insert your namespace once again:

    ```bash
    $ cf ic build -t registry.ng.bluemix.net/<namespace>/app .
    ```
9. Run the app that you just built, linking the mongodb container.

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

Congratulations on completing this tutorial.

# Deploying to IBM Containers

In this tutorial you will take the same sample application used in the [Working with Related Containers](../compose) tutorial and deploy it to the IBM Containers service running on IBM Bluemix.

To complete this tutorial you will need an IBM ID registered with IBM Bluemix. If you do not have an IBM ID, or have not signed up for Bluemix, and wish to complete this section of the lab then sign up for a [free 30-day trial](https://console.ng.bluemix.net/registration/). You will then need the [Cloud Foundry CLI](http://docs.cloudfoundry.org/cf-cli/install-go-cli.html) with the [IBM Containers plug-in](https://console.ng.bluemix.net/docs/containers/container_cli_cfic.html) installed.

1. Log in IBM Bluemix with this command:

    ```bash
    $ bluemix login -sso https://api.ng.bluemix.net
    ```
    1. You will then be prompted to get a one time authentication code at https://iam.ng.bluemix.net/oidc/passcode. Follow this link and type the passcode on screen into the terminal.
    1. Then select the account you want to log on with.

2. If you haven't use IBM Containers before, you need to specify a unique registry namespace to use with your account. Firstly, you will need to install the container registry plugin if you haven't already, with this command:

    ```bash
    $ bx plugin install container-registry -r Bluemix
    ```

3. After this, run this command to select the org:

    ```bash
    $ bx target -cf
    ```

4. From there, use this command to add a namespace to create your own image repository. Replace <my_namespace> with your chosen namespace.

    ```bash
    $ bx cr namespace-add <my_namespace>
    ```

5. If you are not sure if you already have a namespace, use this to list the current namspaces you have created.

    ```bash
    $ bx cr namespace-list
    ```

6. Log your local Docker daemon into the IBM Bluemix Container Registry:

    ```bash
    $ bx cr login
    ```

7. Change to the `bluemix` directory containing this README.

    ```bash
    $ cd bluemix
    ```

8. Build the docker image from the Dockerfile in this directory

    ```bash
    $ docker build -t mongotest .
    ```

9. Tag the image with the destination we will be pushing to (your private registry).

    ```bash
    docker tag mongotest registry.ng.bluemix.net/<my_namespace>/mongo-private-registry:latest
    ```

10. Push the image, where <my_namespace> is the namespace you created earlier in this tutorial.

    ```bash
    docker push registry.ng.bluemix.net/<my_namespace>/mongo-private-registry:latest
    ```

11. Verify your image is in your private registry.

    ```bash
    bx cr image-list
    ```

12. Look at the Dockerfile that will be used to build the application image using the following command. Notice the base image being used is the `ibmliberty` image that is published in the IBM Containers registry.

    ```bash
    $ cat Dockerfile
    ```
    1. In order to make the image into a container and deploy it on bluemix, you will need to create a kubernetes cluster with bluemix.

13. In the top right corner of console.bluemix.net , go to Catalog > Containers > Kubernetes cluster.

14. Select standard for the cluster type and create the cluster.

15. Afterwards, initialise the cluster with your CLI, using this command (US South is used in this demo, but other regions can be used by changing the url after host):

    ```bash
    $ bx cs init --host https://eu-central.containers.bluemix.net
    ```

16. List all of the clusters to check that your cluster is working and available.

    ```bash
    $ bx cs clusters
    ```

17. Set the cluster you created as the context for this session. Complete these configuration steps every time that you work with your cluster.

    1. Get the command to set the environment variable and download the Kubernetes configuration files.

        ```bash
        $ bx cs cluster-config <cluster_name_or_id>
        ```

    2. After downloading the configuration files, a command is displayed that you can use to set the path to the local Kubernetes configuration file as an environment variable. Copy and paste the command that is displayed in your terminal to set the KUBECONFIG environment variable.

    3. Verify that the KUBECONFIG environment variable is set properly.

        ```bash
        $ echo $KUBECONFIG
        ```
        1. Output:
            1. /Users/<user_name>/.bluemix/plugins/container-service/clusters/<cluster_name>/kube-config-prod-dal10-<cluster_name>.yml

18. After this, verify that kubectl runs properly with your cluster by entering this command. A client and server version should appear if configured correctly.

    ```bash
    $ kubectl version --short
    ```

19. Now to deploy the app, use this command to run the configuration script included in this directory to the cluster's context.

    ```bash
    $ kubectl apply -f <deployment script location>
    ```
    1. When writing the deployment script, ensure that the node port is set in the service, and that the app name for the service and deployment are the same. Use the registry.ng.bluemix.net/<my_namespace>/<image_name>:<image_tag> set earlier in the containers image link.

20. Go to the public IP found for your cluster in the bluemix containers dashboard, append the port number on the end of the IP and verify that the service works as intended.

## Congratulations

Congratulations on completing this tutorial.

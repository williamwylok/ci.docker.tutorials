# WebSphere Application Server and Docker Tutorials

This project contains hands-on tutorials that walk through the use of WebSphere Application Server with Docker. The tutorials assume that you are running [Docker Toolbox](https://www.docker.com/products/docker-toolbox/) (1.10 or later) on Windows or OS/X. Begin by cloning this repository on to your host file system. Each of the tutorials may be completed independently but the following is a suggested order:

1. [Running WebSphere Liberty Containers](liberty) introduces the basics of starting a Docker container with WebSphere Application Server Liberty
2. [Adding an Application](app) looks at the different ways in which an application can be run on top of WebSphere Liberty running in a container
3. [Working with Related Containers](compose) covers linking together related containers and using Docker Compose to simplify their management
4. [Deploying to IBM Containers](bluemix) shows how to build and deploy to the IBM Containers service running on IBM Bluemix

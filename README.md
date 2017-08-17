# Docker Compose Redhawk

This repository contains a docker-compose.yml file and supporting files for deploying a full Redhawk application on a Docker Stack.

## Prerequisites

### Docker

This guide requires that you have installed Docker CE and Docker Compose on each machine that you are using to launch the Redhawk application. For the purposes of following along with the command syntax in this guide, you should have a Linux-based operating system and the `docker` daemon should be configured to run with administrative privileges. Docker CE version `17.03.1-ce` and Docker Compose version `1.15.0` were used for this guide.

* See the guide [here](https://docs.docker.com/engine/installation/) to install Docker CE.
* See the guide [here](https://docs.docker.com/engine/installation/linux/linux-postinstall/#manage-docker-as-a-non-root-user) to configure the `docker` daemon to have administrative privileges.
* See the guide [here](https://docs.docker.com/compose/install/) to install Docker Compose.

### Images

The following [Redhawk](http://geontech.com/redhawk-sdr/) images are used by the docker-compose.yml file. These images should either be available locally (see our post [here](http://geontech.com/introduction-docker-redhawk/) on how to build them), you should have internet access to the Geon Technologies' Docker Hub [registry](https://hub.docker.com/u/geontech/dashboard/), or you should have access to another registry that contains these images.

* `geontech/redhawk-omniserver`
* `geontech/redhawk-domain`
* `geontech/redhawk-gpp`
* `geontech/redhawk-development`

In addition to the Redhawk images, we are also using the visualization tool from the Docker Hub Samples [registry](https://hub.docker.com/u/dockersamples/dashboard/). This image is not required for the run-time capabilities of the Redhawk application, but it is useful to view the Docker Containers that have been deployed to Swarm Nodes.

* `dockersamples/visualizer`

## Setting up the Docker Swarm

A Docker Swarm is a distributed computing cluster that is composed of one or more machines, or Swarm Nodes. The Swarm allows us to deploy Redhawk Docker Containers to multiple machines, achieving the goal of a distributed Software-Defined Radio processing environment.

> Note: If you only have a single machine to create a Swarm and deploy your Redhawk application, that's OK! Follow the instructions below to "Initialize the Swarm Manager" and skip the "Add Workers to the Swarm" section.

### Initialize the Swarm Manager

The Swarm Manager is an active Swarm Node that controls all actions for the Swarm, most notably deploying and removing a Docker Stack. Only one Manager can exist per Swarm, and the Manager must be initialized before Workers can join the Swarm. Use the following terminal command to initialize the Swarm Manager:

    $ docker swarm init

> Note: If you receive the error "Error response from daemon: --live-restore daemon configuration is incompatible with swarm mode", remove the "live-restore":true from your `/etc/docker/daemon.json` configuration file.

After executing the initialization command, the terminal will print a message with instructions on how to add a Worker to the Swarm.  The message will look like the following:

    docker swarm join --token `<token>` `<ip_address>`:2377

If you intend to create a Swarm with multiple machines, remember the `<token>` and the `<ip_address>` that were printed to the screen above. If you have only one machine to create a Swarm, skip over the "Add Workers to the Swarm" section.

### Add Workers to the Swarm

A Swarm Worker is a passive Swarm Node that runs Docker Containers based upon the deployments from the Swarm Manager. Once the Swarm Manager is initialized, you much join each Swarm Worker to the Swarm.

> Remember: You must have Docker CE and Docker Compose installed on each machine in the Swarm, including Swarm Workers!

Use the following command within the terminal of each machine that you would like to become a Swarm Worker:

    docker swarm join --token `<token>` `<ip_address>`:2377

That's it! The machine has been registered as a Swarm Worker, and no more steps need to be performed on the machine!

## Deploy the Redhawk application on the Docker Stack

In Docker Compose language, each Docker Container that makes up the Redhawk application is considered a Docker Service. A collection of these services defined in a Docker Compos `.yml` file is considered a Docker Stack.

To deploy the Redhawk application, execute the following terminal command on the <b>Swarm Manager</b>:

    $ docker stack deploy -c docker-compose.yml redhawk

The terminal will print messages with the names of the Docker resources that it has created. It should look something like this:

    Creating network redhawk_default
    Creating service redhawk_omniserver
    Creating service redhawk_domain
    Creating service redhawk_gpp
    Creating service redhawk_visualizer

The Redhawk application is now deployed as a Docker Stack on your Docker Swarm! To view the Containers and the Swarm Nodes on which they were deployed, execute the following command:

    $ docker stack ps redhawk

To have a visual perspective of the Docker Containers and Swarm Nodes, open a web browser and navigate to http://`<ip_address>`:8080/ , using the `<ip_address>` from the Swarm Manager from above.

You are now ready to implement SDR waveforms in your distributed computing Redhawk application! [Let us know](https://geontech.com/contact-us/) how we can help you with all your Redhawk and SDR needs!

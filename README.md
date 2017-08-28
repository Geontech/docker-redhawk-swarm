# Docker Redhawk Swarm

This repository contains Docker Compose files for deploying a full [Redhawk](http://geontech.com/redhawk-sdr/) application on a Docker Stack.

## Prerequisites

### Docker

This guide requires that you have installed Docker CE and Docker Compose on each machine that you are using to launch the [Redhawk](http://geontech.com/redhawk-sdr/) application. For the purposes of following along with the command syntax in this guide, you should have a Linux-based operating system and the `docker` daemon should be configured to run with administrative privileges. Docker CE version `17.06.0-ce` and Docker Compose version `1.15.0` were used for this guide.

> Note: It is recommended that you use the same versions of Docker CE and Docker Compose on all machines within your Docker Swarm. Mixing versions could cause erratic behavior.

* See the guide [here](https://docs.docker.com/engine/installation/) to install Docker CE.
* See the guide [here](https://docs.docker.com/engine/installation/linux/linux-postinstall/#manage-docker-as-a-non-root-user) to configure the `docker` daemon to have administrative privileges.
* See the guide [here](https://docs.docker.com/compose/install/) to install Docker Compose.

### Docker Images

#### Redhawk Images

The following Redhawk images are used to create the Docker Stacks. These images are pulled from Geon Technologies' Docker Hub [registry](https://hub.docker.com/u/geontech/dashboard/). Internet access to this registry is all you need!

* `geontech/redhawk-omniserver`
* `geontech/redhawk-domain`
* `geontech/redhawk-gpp`
* `geontech/redhawk-development`

#### Visualizer Image

In addition to the Redhawk images, we are also using the visualization tool from the Docker Hub Samples [registry](https://hub.docker.com/u/dockersamples/dashboard/). This image is not required for the run-time capabilities of the Redhawk application, but it is useful to view the Docker Containers that have been deployed to Swarm Nodes.

* `dockersamples/visualizer`

#### Using a Local Registry

If your machines do not have internet access or if you want to build your own custom Redhawk images, you must set up a local Docker registry server. Setting up a local registry is outside the scope of this guide, but here are a few notes to help you get started:

* Setup a local Docker registry server using the guide [here](https://docs.docker.com/registry/deploying/)
* Use our guide [here](http://geontech.com/introduction-docker-redhawk/) to build the Redhawk images.
* Each image must be tagged with a prefix containing the hostname and port of the local registry  instead of `geontech/`, and then the images may be pushed to the local registry. The following commands give an example of this process:

```
    $ docker service create --name registry --publish 5000:5000 registry:2
    $ docker tag geontech/redhawk-omniserver computer1:5000/redhawk-omniserver
    $ docker push computer1:5000/redhawk-omniserver
```

* Update the image names in `rh.yml` and `rhide.yml` to have the prefix mentioned above. For example, `geontech/redhawk-omniserver` will have to be changed to `computer1:5000/redhawk-omniserver`.
* Each machine will need the `"insecure-registries":[“computer1:5000"]` key added to `/etc/docker/daemon.json` if your local registry server doesn't have valid SSL certificates. If you want use valid SSL certificates, see our guide [here](http://geontech.com/using-letsencrypt-ssl-internally/) on setting up an SSL server with [Let's Encrypt](https://letsencrypt.org/).

## Setting up the Docker Swarm

A Docker Swarm is a distributed computing cluster that is composed of one or more machines, or Swarm Nodes. The Swarm allows us to deploy Redhawk Docker Containers to multiple machines, achieving the goal of a distributed Software-Defined Radio processing environment!

> Note: If you only have a single machine to create a Swarm and deploy your Redhawk application, that's OK! Follow the instructions below to "Initialize the Swarm Manager" and skip the "Add Workers to the Swarm" section.

### Initialize the Swarm Manager

The Swarm Manager is an active Swarm Node that controls all actions for the Swarm, most notably deploying and removing a Docker Stack. A Manager must be initialized before Workers can join the Swarm. Use the following terminal command to initialize the Swarm Manager:

    $ docker swarm init

After executing the initialization command, the terminal will print a message with instructions on how to add a Worker to the Swarm.  The message will look like the following:

    docker swarm join --token <token> <ip_address>:2377

If you intend to create a Swarm with multiple machines, remember the `<token>` and the `<ip_address>` that were printed to the screen above. If you have only one machine to create a Swarm, skip over the "Add Workers to the Swarm" section.

### Add Workers to the Swarm

A Swarm Worker is a passive Swarm Node that runs Docker Containers based upon the deployments from the Swarm Manager. Once the Swarm Manager is initialized, you much join each Worker to the Manager.

> Remember: You must have Docker CE and Docker Compose installed on each machine in the Swarm, including Swarm Workers!

Execute the following command within the terminal of each machine that you would like to become a Swarm Worker, using the `<token>` and `<ip_address>` from the Swarm Manger initialization:

> Note: If you forgot the token that was generated when initializing the Swarm Manager, return to the Manager terminal and execute the `docker swarm join-token worker` command to view the token and IP address of the Manager.

    $ docker swarm join --token <token> <ip_address>:2377

That's it! The machine has been registered as a Swarm Worker, and no more steps need to be performed on the machine!  

To double check that the Worker has joined the Swarm, execute the following command from the terminal of the Swarm Manager:

    $ docker node ls

The Worker you just added should be listed along with the Manager.

## Deploy the Redhawk application on the Docker Stack

In Docker Compose language, each Docker Container that makes up the Redhawk application is considered a Docker Service. A collection of these services defined in a Docker Compose `.yml` file is considered a Docker Stack.

To deploy the Redhawk application, execute the following command from the terminal of the <b>Swarm Manager</b>:

    $ docker stack deploy -c rh.yml rh

The terminal will print messages with the names of the Docker resources that it has created. It should look something like this:

    Creating network rh_sdrnet
    Creating service rh_omniserver
    Creating service rh_domain
    Creating service rh_gpp
    Creating service rh_visualizer

The Redhawk application is now deployed as a Docker Stack on your Docker Swarm! To view the Containers and the Swarm Nodes on which they were deployed, execute the following command:

    $ docker stack ps rh

To have a visual perspective of the Docker Containers and Swarm Nodes, open a web browser and navigate to http://`<ip_address>`:8080/ , using the `<ip_address>` from the Swarm Manager from above. You are now ready to implement SDR waveforms in your distributed computing Redhawk application!

## Developing in the Redhawk application

A separate Docker Compose file exists for launching the Redhawk IDE, meaning that a separate Docker Stack will be created for the IDE. Although the Stack is separate, it still shares the same volume and network created by the `rh.yml` file. Therefore, the `rhide.yml` file <b>must</b> be deployed after the `rh.yml` file.

To deploy the Redhawk IDE, execute the following commands from the terminal of the <b>Swarm Manager</b>:

    $ export USER_ID=$(id -u)
    $ docker stack deploy -c rhide.yml rhide

The first command sets up an environmental variable used by the `rhide.yml` file, and the second command launches the IDE on the Swarm Manager and exports its display to that machine.

> Note: Closing the Redhawk IDE window does not remove the Docker Stack. After closing the IDE window, you must remove the Stack using the `docker stack rm rhide` command before deploying `rhide.yml` again.

## Stopping the Redhawk application

If you deployed the Docker Stack that drives the Redhawk IDE, you <b>must</b> remove it before removing the base Redhawk application. Execute the following commands in the terminal of the Swarm Manager to stop the Redhawk IDE Docker Stack:

    $ docker stack rm rhide

To stop the Docker Stack that drives the Redhawk application, execute the following commands in the terminal of the Swarm Manager:

    $ docker stack rm rh

## Stopping the Docker Swarm

To kill the entire Docker Swarm and disconnect all Workers, execute the following command from the terminal of the Swarm Manager:

    $ docker swarm leave --force

To leave the Swarm from an individual Worker, execute the command above from the terminal of the Swarm Worker without the `--force` switch:

    $ docker swarm leave

## Troubleshooting

Here is a list of issues that you may face with their solutions:

* <b>I can connect to my Redhawk Domain, but I don't see all of my Devices!</b> You may need to disable the firewall for all your Swarm Nodes to allow the Device Manager to connect. This behavior is documented in Section 2.3.1 of the [Redhawk Manual](https://redhawksdr.github.io/Documentation/mainch2.html).
* <b>When executing `docker swarm init` I receive the message, "Error response from daemon: --live-restore daemon configuration is incompatible with swarm mode".</b> Remove `"live-restore":true` from your `/etc/docker/daemon.json` configuration file.
* <b>When executing `docker swarm init` I receive the message, "Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on different interfaces ... - specify one with --advertise-addr".</b> Use the following format of the command: `docker swarm init --advertise-addr <ip_address>`.

As always, [let us know](https://geontech.com/contact-us/) how we can help you with all your Redhawk and SDR needs!

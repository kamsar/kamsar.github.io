title: 'Running JSS headless mode in containers, part 1'
date: 2019-09-08 12:00:36
categories: JSS
---

I've been playing with containers lately and as an experiment, containerized [JSS Headless Mode](https://jss.sitecore.com/docs/fundamentals/application-modes#headless-server-side-rendering-mode). Since I had fun doing this, I figured I'd share what I learned. Note that this is my own explorations, and should not be construed as any official statement of container support for JSS, nor is it supported via official Sitecore channels.

## Containers 101: What's a container?

The best way to understand containers quickly is, of course, a meme.

{% asset_img "if it works on your machine, then ship your machine" works.png %}

Another way to think of a container is a lightweight virtual machine. Unlike a VM, a container _shares_ much of its system with the host OS or node. This means that containers:

1. Are much smaller both in disk and memory usage compared to a VM
1. Do not provide as strong of an isolation from the host as a VM
1. Are more easily _based on_ a standard distribution. For example in this post we won't be building a container from scratch; we will take the standard `node` container and deploy JSS to it - thus, we offload the maintenance of the base container to the Node maintainers, and we take on the maintenance of only our app.

Containers have become incredibly popular as a way to build and deploy applications because of their consistency and low resource usage. Especially as more applications take on more server-based dependencies (i.e. microservice architectures, or even a traditional app that may need a database, search service, etc), containers provide a reasonable way to replicate such a complex IT infrastructure on a developer machine in the same way that it runs in production - without each developer needing to have a 1TB RAM, 28-core server to run all those virtual machines.

So with that in mind, what if we wanted to containerize Sitecore JSS' headless mode host?

> Note: we're only containerizing the JSS SSR host in this post; the rest of the Sitecore infrastrucure would still need to be deployed traditionally.

## Creating a JSS Docker container

If you're planning to follow along at home with this build, note that you'll need to install [Docker Desktop](https://hub.docker.com/?overlay=onboarding) in order to be able to locally build and run the containers. You may also need to enable virtualization in your UEFI, if it's off, or potentially for Windows also enable Hyper-V and Containers features at an OS level. Consult the Docker docs for help with that :)

When you create a container, there are three main tasks:

### Determine the base container to build from

Containers are built on top of other containers in an efficient and lightweight way. This means that for example, your container might start with a Windows Server container, or an Ubuntu container...or it might start from a Node container, that was based on an Debian container. You get the idea - containers, like ogres or '90s software architecture, have layers. Each layer is built as a diff from the underlying layer. When you make a container, you're adding a layer.

In our case, JSS headless SSR is a Node-based application, so we will choose the [Node container](https://hub.docker.com/_/node) as our base.

### Define the `Dockerfile`

The [dockerfile](https://docs.docker.com/engine/reference/builder/) is a file named `Dockerfile` that defines how to create your container. It defines things like:

* What your base container is (`FROM node:lts`)
* How to modify the base container to turn it into your container (scripts and file copying)
* Defaults, like which TCP/UDP ports the container can expose

In our case we want to start from the [`node` container](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/):

    FROM node:lts

Then we want to tell Docker how to deploy our JSS app on top of the Node container. We do this by telling it which files we want to copy into the container image and where to put them, as well as any commands that need to be run to complete the setup:

    # We want to place our app at /jss on the container filesystem
    # (this is a fairly arbitrary choice; 
    # use something app-specific and don't use '/')
    # Subsequent commands and copies are relative to this directory.
    WORKDIR /jss

    # Specify the _local_ files to copy into the container;
    # in this case a copy of the headless SSR proxy: https://github.com/Sitecore/jss/tree/dev/samples/node-headless-ssr-proxy
    COPY ./node-headless-ssr-proxy /jss

    # Run shell commands _inside the container_ to set up the app;
    # in this case, to install npm packages for the headless Node app.
    # NOTE: the container is built on the Docker server, not locally!
    # Commands you run here run inside the container, and thus 
    # cannot for example reference local file paths!
    RUN npm install

    # To run JSS in headless mode, we also need to deploy 
    # the JSS app's server build artifacts into the container
    # for the headless mode proxy to execute. This is another copy.
    COPY my-jss-app-name/dist /jss/dist/my-jss-app-name

    # When the container starts, we have to make it do something
    # aside from start - in this case, start the JSS app.
    # The command is run in the context of the WORKDIR we set earlier.
    ENTRYPOINT npm run start

    # The JSS headless proxy is configured using environment variables,
    # which allow us to configure it at runtime. In this case,
    # we need to configure the port, app bundle, etc
    ENV SITECORE_APP_NAME=my-jss-app-name

    # Relative to /jss path to the server bundle built by the JSS app build
    # Note: this path should be identical to the path deployed for integrated
    # mode, so that path references work correctly.
    ENV SITECORE_JSS_SERVER_BUNDLE=./dist/${SITECORE_APP_NAME}/server.bundle.js

    # Hostname of the Sitecore instance to retrieve layout data from.
    # host.docker.internal == DNS name of the docker host machine, 
    # i.e. to hit non-container localhost Sitecore dev instance
    ENV SITECORE_API_HOST=http://host.docker.internal
    ENV SITECORE_API_KEY=GUID-VALUE-HERE
    # Enable or disable debug console output (dont use in prod)
    ENV SITECORE_ENABLE_DEBUG=false
    # Set the _local_ port to run JSS on, within the container
    # (this does not expose it publicly)
    ENV PORT=3000

    # Tell Docker that we expose a port, but this is for documentation;
    # the port must be mapped when we start the container to be exposed.
    EXPOSE ${PORT}

### Build the container

Once we have defined the steps necessary to create the container image, we need to _build_ the container. Building the container:

* Collects all the files in the `Dockerfile` directory and uploads them to the Docker host (unless listed in a `.dockerignore` file)
* Acquires the base image, if it's not already on the Docker host
* Creates a container based on the base image and starts it
* Executes your `Dockerfile` script within the container to configure it
* Captures your Docker image and stores it for reuse

> The `Dockerfile` does not execute locally, so make sure you don't make that assumption when using `EXEC` directives; execution also occurs within the container being built, so it occurs in the context of the container (in this case, Debian) and the dependencies that are part of the container.

To build your JSS container, within the same folder as your `Dockerfile` run:

    docker build -t your-image-name .

Once the build is done, you can find your image on Docker using:

    docker images

## Using the JSS Docker container

Up to this point we have collected and built the container, but nothing has been run. To create a new instance of your container and start it up, run

    docker run -p 3000:3000 --name <pick-a-name-for-container-instance> <imagename>

The `-p` maps your localhost port 3000 to the container port 3000 (which we specified the Node host to run on previously using an environment variable).

Once you start the container, visiting `http://localhost:3000` should run the app in the JSS headless host container. 

### Container Debugging Tips

* Viewing running containers - list running containers using the `docker ps` command. If a container was started without an explicit `--name`, this can help find it.
* Opening a shell in a container - to run diagnostic shell commands, you can open a root shell to a running container. The `docker exec` command lets you run commands, including starting a shell - for example, `docker exec -it <container-name> bash`. The `-it` says you want an interactive TTY (in other words an ongoing shell, not a one-off command execution and exit)

## What's Next?

In this post, we've created and run a Docker container of the JSS headless mode. This works great for a single container, but for production scenarios we would likely need to orchestrate _multiple_ instances of the container to handle heavy load and provide redundancy. Next time, we will improve our container build script using a build container, then finally the series will end with orchestrating the container using Kubernetes.
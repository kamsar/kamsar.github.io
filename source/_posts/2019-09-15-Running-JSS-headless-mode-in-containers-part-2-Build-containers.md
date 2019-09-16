title: 'Running JSS headless mode in containers part 2: Build containers'
date: 2019-09-15 19:14:04
categories: JSS
---

In [part 1](/index.php/2019/09/Running-JSS-headless-mode-in-containers-part-1/) of this series, we investigated containerizing the JSS headless mode host. But there are some issues with that basic implementation:

* We still have to pre-compile the JSS app to extract the server bundle, before building the container, and this compile step is not necessarily consistent with the production container environment since it depends on the host environment (different Node versions, OSes, etc are possible)
* The standard [Node container](https://hub.docker.com/_/node) is pretty large (900MB), and though there are reasonable reasons for this, something smaller would be nice for production
* Because the API key and Sitecore hostname are baked into the server bundle at build time, it would be necessary to maintain a container image for each environment

So, let's fix this by using a _build container_ to create a standardized build environment where we can make our server bundle. Basically, 

{% asset_img "dawg.jpg" %}

## Build Containers 101

When a `Dockerfile` executes, the state of each intermediate step is stored so that it need not be repeated every time the image rebuilds. But more importantly, you can also create intermediate _build containers_ that live only long enough to perform a build step, then get thrown away except for some artifacts you want to persist on to some future part of the container build. In the case of our JSS image, the idea is something like this:

* Create a full Node container
* Copy the JSS app into it and run `jss build`
* Create a lightweight Node container
* Deploy the artifacts from the full Node container and the JSS headless app into it

The build container that we use to build the JSS app in is thrown away after the build occurs, leaving only the lightweight production container with its artifacts. In this case, thanks to switching from `node:lts` to `node:lts-alpine` as the base container, the built container size shrunk from 921MB to 93MB.

> Note that because the base image is stored as a diff, the image size reduction affects the _initial download time_ of the image on a new host, but once the `node:lts` image is cached it really only changes the amount of static disk space consumed.

Adding a build container step involves adding a few lines to the top of the `Dockerfile` from [part 1](/index.php/2019/09/Running-JSS-headless-mode-in-containers-part-1/):

    # Create the build container (note aliasing it as 'build' so we can get artifacts later)
    FROM node:lts as build
    # Install the JSS CLI
    RUN npm install -g @sitecore-jss/sitecore-jss-cli
    # Set a working directory in the container, and copy our app's source files there
    WORKDIR /build
    COPY jss-angular /build
    # Install the app's dependencies
    RUN npm install
    # Run the build
    RUN npm run build

    # Now, we need to switch contexts into the final container
    # lts-alpine is a lightweight Node container, only 90MB
    # When we switch contexts, the build container is supplanted
    # as the context container
    FROM node:lts-alpine

    # ...
    # When we copy the app's source into the final container
    # we need to use --from=[tag] to get the files from our build container
    # instead of the local disk
    COPY --from=build /build/dist /jss/dist/${SITECORE_APP_NAME}

## Tokenizing the Server Bundle

To solve the issue of the Sitecore API URL and API Key being baked into the server and browser bundles by webpack during `jss build`, we need to use tokenization. These values really do need to be baked into the file at some point, because the browser that executes them does not understand environment variables on your server or how to replace them - but, we should not need to re-run webpack every time a container starts up either.

We can work around this by baking specific, well-known tokens into the bundle files and then expanding those tokens when the container starts from environment variable values. The approach works something like this:

* When the build container builds the JSS app, we force it to use specific well-known token values for the API host and API key, such as `%sitecoreApiHost%`
* We move all the build output from the build container from `*.js` files to `*.base` files. This means the container itself does not contain any JS in its `/dist`. This is necessary so the container can generate the final files each time it starts up. (Since the same image can start many times with different environment variables present, it has to 'rebake' the JS each time)

Doing this is a bit harder than just doing the build container. First, during the container build in the `Dockerfile`:

    # Before the build container runs the `build` command,
    # we need to set specific API key and host values to bake
    # into the build artifacts to replace later.
    RUN jss setup --layoutServiceHost %layoutServiceHost% --apiKey 309ec3e9-b911-4a0b-aa8d-425045b6dcbd --nonInteractive

    RUN npm run build

    # After the build container runs the `build` command,
    # we need to move all the .js files it emitted to .base files
    RUN find dist/ -name '*.js' | xargs -I % mv % %.base

With the updated `Dockerfile` in place, the container we build will now have `.base` files ready to specialize into the running configuration when the container starts up. But without any changes to the image itself, it would fail because we can't run an app using `.base` files! So we need to add a little script to the `node-headless-ssr-proxy` to perform this specialization when it starts up inside a container. The specialization process:

* Copy all `.base` files to `.js` file of the same name (make a runtime copy to use in the browser)
* Search & replace the well-known tokens in the `.js` files with the current runtime environment variables
* Start the JSS headless proxy, which will now use the generated `.js` files and run normally

I used `bootstrap.sh` for the script name, but any name is fine.

```bash
find dist/ -name '*.base' | while read filename; do export jsname=$(echo $filename | sed -e 's|.base||'); cp $filename $jsname; sed -i -e "s|%layoutServiceHost%|$SITECORE_API_HOST|g" -e "s|309ec3e9-b911-4a0b-aa8d-425045b6dcbd|$SITECORE_API_KEY|g" $jsname; done
```

This script is a rather hard to read one-liner so let's piece it out to understand it:

* `find dist/ -name '*.base' | while read filename` - Finds `*.base` files anywhere under `dist`, and reads each found filename into `$filename` in a loop body
* `do export jsname=$(echo $filename | sed -e 's|.base||')` - Sets `$jsname` to the name of the found file, with the extension changed from `.base` to `.js`
* `cp $filename $jsname` - copies the `.base` file to the equivalent path, but using the `.js` extension instead
* `sed -i -e "s|%layoutServiceHost%|$SITECORE_API_HOST|g" -e "s|309ec3e9-b911-4a0b-aa8d-425045b6dcbd|$SITECORE_API_KEY|g" $jsname` - uses `sed` to perform a regex replace on the known values we baked into the base file, replacing them with the environment variables (`$SITECORE_API_HOST` and `$SITECORE_API_KEY`) that form the current runtime configuration for those values

Finally we need to get this script to run each time the container starts up. There are several ways we could do this, but I elected to add an npm script to the headless proxy's `package.json`:

```json
  "scripts": {
    "start": "node index.js",
    "docker": "./bootstrap.sh && node index.js"
  },
```

...and then changed the entry point in the `Dockerfile` to call the container entrypoint:

    ENTRYPOINT npm run docker

The final step is to rebuild the container image so we can start it up, using `docker build`.

### Using the tokenized container

The headless proxy Node app has always known how to read environment variables for the Sitecore API host and API key, but those have only applied to the **SSR** execution not the browser-side execution. With the modifications we've made, setting those same environment variables will now also apply to the browser. Doing this with Docker is quite trivial when booting the container, for example:

```bash
docker run -p 3000:3000 --env SITECORE_API_KEY=[yourkey] --env SITECORE_API_HOST=http://your.site.core [container-image-name]
```

## Putting it together

For more clarity, here's the full contents of the `Dockerfile` with all these changes made:

```docker
FROM node:lts as build
RUN npm install -g @sitecore-jss/sitecore-jss-cli
WORKDIR /build
COPY jss-angular /build
RUN npm install
# setup to use static values we'll later replace with env vars
# (for values that are baked into the server bundle)
RUN jss setup --layoutServiceHost %layoutServiceHost% --apiKey 309ec3e9-b911-4a0b-aa8d-425045b6dcbd --nonInteractive
RUN npm run build
# Rename all .js files to .js.base (so we can bootstrap tokens later)
RUN find dist/ -name '*.js' | xargs -I % mv % %.base

FROM node:lts-alpine

ENV SITECORE_API_HOST=http://host.docker.internal
ENV SITECORE_API_KEY=MYKEY
ENV SITECORE_APP_NAME=jss-angular
ENV SITECORE_JSS_SERVER_BUNDLE=./dist/${SITECORE_APP_NAME}/server.bundle.js
ENV SITECORE_ENABLE_DEBUG=false
ENV PORT=3000

WORKDIR /jss
COPY ./node-headless-ssr-proxy /jss
COPY --from=build /build/dist /jss/dist/${SITECORE_APP_NAME}
RUN npm install

ENTRYPOINT npm run docker
EXPOSE ${PORT}
```

In this episode, we have improved the JSS headless container build process by running all of the build inside containers for improved repeatability and tokenized the browser JS bundles so that the same container can be deployed to many environments with different API hosts without needing a rebuild. What's next? Orchestrating multiple instances with Kubernetes.
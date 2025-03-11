# Building Secure Container Images - Workshop Guide

## Index

- [Prerequisites](#prerequisites)
- [Basic Dockerfiles](#basic-dockerfiles)
- [Layered](#layered)
- [Vulnerability analysis between simple and multi-stage builds](#vulnerability-analysis-between-simple-and-multi-stage-builds)
- [Alpine base - Is it always usefull?](#alpine-base---is-it-always-usefull)
- [Distroless](#distroless)
- [Distroless - Google](#distroless---google)
- [Dive](#use-dive-to-visualize-the-layers)
- [Add security mechanisms](#add-security-mechanisms)
- [Exercise](#exercise)
- [Clean Up](#clean-up)
- [Extension](#extension)

## Prerequisites

Ensure you have the following installed:
- **Docker** (or any other cri-compatible)
- **Trivy** (To scan vulnearbities, it can be run as a container)
- **Dive** (to inspect image layers)

### Installations
- [Get Docker](https://docs.docker.com/get-started/get-docker/)
- [Get Trivy](https://github.com/aquasecurity/trivy?tab=readme-ov-file#get-trivy)
- [Get Dive](https://github.com/wagoodman/dive?tab=readme-ov-file#installation)
---

## Basic Dockerfiles

Let's move to the first basic folder: `cd basic`

In this folder we see our initial application, the Dockerfile here is quite simple.

```
# use ubuntu as base image
FROM ubuntu

# copy the source code
COPY hello.c hello.c

# install build-essential package to compile the source code
RUN apt update
RUN apt install -y build-essential

# Compile and generate binary
RUN gcc -o helloWorld hello.c

# Run the program
ENTRYPOINT ["/helloWorld"]
```

We can build our image, inside the folder

`docker build . -t devopsdays:basic1`

It will take some time to build (depending on your laptop, for me it's around **1 minute**); You can already see how the stage that it is taken the most time is the installation of the libraries needed to compile. After that, we can be sure that it works:

`docker run --rm devopsdays:basic1`
> Hello World!

Let's go back and move to the other basic folder

`cd ..` ` cd basic2`

This file it's almost the same, but with a difference, we have moved switch the order of two of the instructions:

```
# use ubuntu as base image
FROM ubuntu 

# install build-essential package to compile the source code
RUN apt update
RUN apt install -y build-essential

# copy the source code
COPY hello.c hello.c

...
```

Why? Because of how we saw that this was the instruction that takes most of the time, and due to the layered nature of containers explained before. Let's first build it

`docker build . -t devopsdays:basic2` (it will be considerable faster)

Now, if we edit both **hello.c** in the two basic folders, and build then again, you will see how one is much more faster.

## Layered

Another technique is the use of two (or more) base images. One will compile the source code while the other one will actually run it

`cd ..``cd layered`
```
# use ubuntu as base image
FROM ubuntu as build-env

# install build-essential package to compile the source code
RUN apt update && apt install -y build-essential

# copy the source code
COPY hello.c hello.c

# Compile and generate binary
RUN gcc -o helloWorld hello.c

# FROM alpine for an even greater size reduce
FROM ubuntu
    
# copy binary executable to new layer
COPY --from=build-env ./helloWorld ./helloWorld 

# Run the program
ENTRYPOINT ["/helloWorld"]
```

As you see, we name the first FROM UBUNTU as the build enviroment. Later on, we tell docker to import from this stage **only** the file(s) that we want.

Again, let's build it 

`docker build . -t devopsdays:layered1`

Now, we can retrieve the information of the images and see the differences:

`docker images devopsdays`
|  REPOSITORY |  TAG | SIZE  |
| ------------ | ------------ | ------------ |
|  devopsdays |  basic1 | 491MB  |
|  devopsdays |  basic2 | 491MB   |
|  devopsdays |  layered | 101MB   |

Our new layered is 4 times lighter, this is a good job, not only because it uses less disk space (leading to faster deployments), but because it reduces the **attack surface**. We want, whenever possible, as mininal images as possible.

## Vulnerability analysis between simple and multi-stage builds

One tool to see how our new image is safer is trivy

`trivy image devopsdays:layered1`

If we compare between our two versions, this are the results:

|  Technique |  Total vuln | 
| ------------ | ------------ |
|  Basic |  952 | 
|  Layered |  23 |

We can clearly see how our second option, that does not include the modules needed to build, is in less risk of being exploited. 

## Alpine base - Is it always usefull?

There is another base OS we can use, which is quite well known inside this world, that is the images **alpine**. They are much more lightweight.

If we compile **layered2** we will get an image of 10 times smaller! However, don't get tricked, as alpine as some **serious** limitations.

`docker build . -t devopsdays:layered2`
`docker images devopsdays:layered2`

When dealing with containers, one important concept is determinism, that means that subsequent build of the same Dockerfile will yield the same results. We archieve that using tags (ubuntu:24.04). However alpine, being a minimalistic OS actually removes old versions of applications from their repositories, thus defeating the whole concept of determism.

So, for example, if our application needs a specific version of a library to work. And updating that library will mean failure to build. Then, be carefull when using Alpine. The official recomendation from them is to host your own mirror repository

> [Link to the GitLab discussion](https://gitlab.alpinelinux.org/alpine/abuild/-/issues/9996)

So when using Alpine based images you need to be carefull, they are usefull as long as you understand it's limits.

## Distroless

Let's see how to manually craft a distroless, following our example

> Note that, although I am aware that I can compile in C to include the libraries, thus removing the need to manually copy them, the idea is to show how the SCRATCH images are fully empty by default and what that implies.

`cd ..` `cd distroless`
```
# use distroless container to run the program
FROM scratch

# copy the required libraries to run the program
COPY --from=build-env  /lib/aarch64-linux-gnu/libc.so.6 /lib/aarch64-linux-gnu/libc.so.6
COPY --from=build-env  /lib/ld-linux-aarch64.so.1 /lib/ld-linux-aarch64.so.1

# copy the /etc/passwd 
COPY --from=build-env  /etc/passwd /etc/passwd

# tmp directory
COPY --from=build-env  /tmp /tmp

# copy binary executable to new layer
COPY --from=build-env ./helloWorld ./helloWorld 

# Run the program
ENTRYPOINT ["/helloWorld"]
```

We can built and verify that it works

`docker build . -t devopsdays:distroless`

`docker run --rm devopsdays:distroless`

You can try to play around and see what happens when you comment out some of the libraries.

## Distroless - Google

The generous people at Google already provide us with some distroless images wich reduces some of the burden of copying things over.
They are bigger than doing them ouselves **FROM SCRATCH**. But this ones are actually production-ready.

> And actually, Kubernetes uses this images for running it's internal componentes

`cd ..``cd distrolessgoogle`
```
# use distroless container to run the program
FROM gcr.io/distroless/cc
    
# copy binary executable to new layer
COPY --from=build-env ./helloWorld ./helloWorld 

# Run the program
ENTRYPOINT ["/helloWorld"]
```
`docker build . -t devopsdays:distrolessgoogle`

A quick analysis between the image size (`docker images devopsdays`) and vulnerabities (`trivy image devopsdays:XXX`)

|  Technique |  Total vuln | Size |
| ------------ | ------------ | ------------ |
|  Basic |  952 | 491 Mb | 
|  Layered |  23 | 101 Mb |
|  Distroless |  0? | 2 Mb |
|  Distroless Google |  17 | 34.1 Mb |


## Use Dive to visualize the layers

Let's finish off this section using the tool Dive to visualize the difference between the differnet images we have created.

`dive devopsdays:layered1`

`dive devopsdays:distroless`

You can see how every instruction generated a new layer that added a file, and visualy understand the concept of images that come from scratch.

## Add security mechanisms

Lastly, another pivotal concept is to avoid unncesary privileges. So far, all of our containers have been running as root, which is the default behaviour. We want to avoid that for obvious reasons.

We can check it, by running this command:

`docker run --rm --entrypoint "/bin/bash" -it devopsdays:layered`

`root@91bc84c2b139:/# whoami`

> This experiment won't work on our distroless images because they don't include a shell, in tools like Kubernetes you can use epehermal containers to debug this images.

If we add the following instructions:

`cd ..` `cd final`
```
# Create user and set ownership and permissions as required
RUN adduser -D myuser

# copy binary executable to new layer
COPY --from=build-env ./helloWorld ./helloWorld 

USER myuser

# Ensure the binary is executable
RUN chmod +x ./helloWorld

# Run the program
ENTRYPOINT ["/helloWorld"]
```

Note that we have not use chown to change the ownership of the file, that it's a good thing. The owner will still be root so that our new user, if compromise, cannot alter it. Only execute.

`docker build . -t devopsdays:finalubuntu`

`docker run --rm --entrypoint "/bin/bash" -it devopsdays:finalubuntu`

Again, the google distroless provides with a non root tag, you can play around in the Dockerfile uncometing the lines that say so.


## Exercise

Finally, use all the concepts and improve the given image

`cd ..` `cd exercise`


`docker build . -t devopsdays:exercise`
`docker run --rm -p 3000:3000 devopsdays:exercise`

Open [http://localhost:3000](http://localhost:3000)

- [ ] Analize the Dockerfile and modify it so use multistage builds
- [ ] Improve the multistage using google distroless as the foremost image -> [Repository](https://github.com/GoogleContainerTools/distroless)
- [ ] Analize the basic, layered and distroless using both trivy and Dive
- [ ] If you have time, you can make your own distroless using from scratch, tip, you can use `ldd` to find the libraries a binary needs

## Clean Up

docker rmi devopsdays
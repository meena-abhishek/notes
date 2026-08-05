a container is a way to package application with all the necessary dependencies and configuration.

VM, guestOS -> kernel -> libraries -> application
DOCKER, libraries -> application

docker uses multi-architecture images

to build custom-image for multiple architectures use `docker buildx build`

to run linux images, docker uses a linux-vm

`Image  --docker run-->  Container`

`-d` detach container

binding between container ports and host ports is different.

`docker run -phost:container --name redis`

`docker exec -it [container-id/name] /bin/bash`

`docker run` - create a container
`docker start/stop`

private docker repo

containers in same docker network can communicate using just docker names.

`docker network create name`
`docker run --net net_name`

docker compose,
write run commands for multiple containers in a structured way.

```yaml
//mongo.yaml
version:'3'  // docker-compose version
servies:     // containers list
	mongodb:
		image: mongo
		ports:
		 - 2700:5000  // host:container
		environment:
		 - MONGO_USERNAME=admin
		volumes:
		 - db-data:/var/lib/mysql/data
volumes:
	db-data
```

docker compose creates a common network for all these containers
`docker-compose -f mongo.yaml up/down`

![[docker_workflow.png|600]]

dockerfile - blueprint for creating a image

```dockerfile
FROM node:13
ENV USER=admin\
	 PASS=123
RUN mkdir -p /home/app
COPY . /home/app      //executes on host machine copies to container
CMD ["node","server.js"]   //executes "node server.js"
```
`-t` tag

host/anon/named volumes

`docker run -v name:/var/lib/mysql/data`


### Q. Multi-Architecture Manifests
Instead of requiring a different image name for every type of computer, Docker uses **Manifest Lists** (also known as fat manifests).
When you pull an image like `ubuntu:latest` or `node:lts`, Docker doesn't just grab one file. It looks at a manifest list that points to multiple versions of that image built for different architectures. Docker automatically detects your local CPU architecture and pulls the exact version matching your system
### Q. Cross-Platform Emulation (QEMU)
What happens if you are on an Apple Silicon Mac (`arm64`) but absolutely _must_ run an older Docker image that was only built for Intel (`amd64`)?
Docker Desktop integrates **QEMU**, a hosted virtual machine monitor that performs on-the-fly binary translation.
- When you run `docker run --platform linux/amd64 my-intel-image` on an ARM machine, Docker uses QEMU to translate the Intel instructions into ARM instructions.
- _Note:_ While this makes the image completely portable, it comes with a performance cost due to the emulation layer.

The more classic "architecture problem" Docker solved early on was the chaos of different operating systems and environment configurations—famously known as the **"It works on my machine"** dilemma.

- **Kernel Isolation via Namespaces and Cgroups:** Docker leverages Linux kernel features to slice a single operating system into isolated user spaces.
    
- **The Blueprint (Dockerfile):** Everything from the base OS configuration, system libraries, environment variables, down to the exact patch version of a language runtime is codified.
    
- **Immutable Layers:** Because Docker images are built out of read-only, stackable layers, the environment layout is completely frozen. Whether that image is initialized on a Windows laptop, a macOS workstation, or a RedHat enterprise cloud server, the internal software environment remains identical
### Buildx
Buildx solves several advanced packaging problems that the legacy builder couldn't handle easily:
- **Simultaneous Multi-Platform Builds:** You can compile and package an image for `amd64` (Intel/AMD), `arm64` (Apple Silicon/Raspberry Pi), and `386` all in a single command.
- **Scoping Build Environments:** You can create isolated, dedicated containerized environments to run your builds, meaning your local Docker daemon's configuration doesn't interfere with your builds.
- **Advanced Cache Management:** Buildx can export your build cache to external locations—like a remote registry (e.g., Docker Hub, GHCR), a local directory, or an AWS S3 bucket—making CI/CD pipelines incredibly fast.
- **Kubernetes Native Building:** You can point Buildx to a remote Kubernetes cluster or a remote server and run your heavy build workloads there, instead of melting your laptop's CPU.

--- 

Docker provides the ability to package and run an application in a loosely isolated environment called a container

Containers are lightweight and contain everything needed to run the application, so you don't need to rely on what's installed on the host

Kubernetes runs an application as one or more pods (containers). Under normal traffic, it may run a few replicas of the application, such as 3 pods. When traffic increases and CPU or memory usage crosses a threshold, Kubernetes can automatically create additional pods from the same Docker image, for example scaling from 3 to 10 pods. A load balancer then distributes incoming requests across all available pods. When traffic decreases, Kubernetes removes the extra pods to save resources. Because every pod is created from the same Docker image containing the application code, runtime, and dependencies, scaling can happen quickly without manually setting up new servers or reinstalling software.

Docker is lightweight because containers share the host operating system's kernel instead of running a separate guest operating system like virtual machines. This reduces CPU, memory, and storage overhead, allowing many more containers than VMs to run on the same hardware. Containers also start in seconds rather than minutes, making deployment and scaling faster. As a result, organizations can utilize server resources more efficiently, lower infrastructure costs, and run more applications on fewer machines. This makes Docker particularly well-suited for high-density environments and small-to-medium deployments where maximizing resource usage and minimizing operational costs are important.

## [Docker architecture](https://docs.docker.com/get-started/docker-overview/#docker-architecture)

Docker uses a client-server architecture. The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers. The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon. The Docker client and daemon communicate using a REST API, over UNIX sockets or a network interface. Another Docker client is Docker Compose, that lets you work with applications consisting of a set of containers.

![[docker-architecture.webp|550]]

An image is a read-only template with instructions for creating a Docker container.When you change the Dockerfile and rebuild the image, only those layers which have changed are rebuilt

A container is a runnable instance of an image.You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state.

Docker allocates a read-write filesystem to the container, as its final layer. This allows a running container to create or modify files and directories in its local filesystem.

Docker creates a network interface to connect the container to the default network, since you didn't specify any networking options. This includes assigning an IP address to the container. By default, containers can connect to external networks using the host machine's network connection.

![[vm_vs_containers.png|500]]

![[user_vs_kernel_space.png|500]]

`docker compose watch` 
docker automatically copies changed files into the container, no need to rebuild image and restart containers.

the processes running in each container are watching and responding to file changes and 2) the files in your local project directory are shared with the containerized environment, so edits you make locally are automatically synced to the containers.

[Docker Hardened Images (DHI)](https://docs.docker.com/dhi/)


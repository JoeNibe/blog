# Docker Security - Part 1 (Build Time security)

In this blog, we will be looking into docker container build time security. I will be covering the following points in this blog
- Deamonless Builds
- Docker file security
- Storing and deploying images

Readers are expected to have a basic understanding of containers. If you are not familiar with the basics of container please refer this [blog](https://www.freecodecamp.org/news/demystifying-containers-101-a-deep-dive-into-container-technology-for-beginners-d7b60d8511c1/).

## Deamonless Builds:

Docker is one of the most prevalent ways to build and run a container. Running docker commands send an API request to the Docker daemon via Docker socket which then does all the heavy lifting. Any program with access to Docker daemon can send API requests to the daemon. If your organization use a dedicated machine to build container images then any user who can trigger a docker build on the machine can also perform docker run to execute any command on the machine. Docker is also an easy way to [escalate privleges](https://gtfobins.github.io/gtfobins/docker/) on a compromised system.

There are alternative tools that mitigate this secuity risk.
	
		
1. [BuildKit](https://github.com/moby/buildkit) 
	BuildKit is a toolkit for converting source code to build artifacts in an efficient, expressive and repeatable manner. This is an actively managed project and can be used as an alternative for Docker.
2. [pdman](https://podman.io/) 
	Podman is a daemonless container engine for developing, managing, and running OCI Containers on your Linux System. Containers can either be run as root or in rootless mode.
3. [buildah](https://buildah.io/)
	Both, Buildah and Podman are command line tools that work on Open Container Initiative (OCI) images and containers. The two projects differentiate in their specialization.
Buildah specializes in building OCI images. Buildah's commands replicate all of the commands that are found in a Dockerfile. This allows building images with and without Dockerfiles while not requiring any root privileges.
4. [Bazel](https://github.com/bazelbuild/rules_docker)
	Bazel can build many other types of artifact, not just container images. Not only does it not require Docker, but it also prides itself on generating images deterministically so that you can reproduce the same image from the same source.
5. [Kaniko](https://github.com/GoogleContainerTools/kaniko)
	kaniko is a tool to build container images from a Dockerfile, inside a container or Kubernetes cluster. Kniko doesn't depend on a Docker daemon and executes each command within a Dockerfile completely in userspace. This enables building container images in environments that can't easily or securely run a Docker daemon, such as a standard Kubernetes cluster.
6. [img](https://github.com/genuinetools/img)
	img is a sandalone, daemon-less, unprivileged Dockerfile and OCI compatible container image builder.
	
	## Docker file security
	Dockerfile consists of instructions to build a container image. These can also be used to perform various operations on the	image such as installing packages, copy file from the local system etc. An attacker with access to Dockerfile can perform malicious actions such as adding malware to the image, accessing build secrets, enumerating network topology, attacking the build host machine etc. This shows that Dockerfiles need proper access controls. Here are some way to secure the docker file
	1. Base Image
		The FROM instruction indcates the source base image from which the image is build.  Ensure that the image is from a trusted source, arbitary third party images can contain malicious code. It is advisable to use a smaller base image as it will contain lesset code therby reducing the attack surface. Or if possible build the image from scratch or consider using a minimal image such as [distroless](https://github.com/GoogleContainerTools/distroless).
	2. Multi stage builds
		This is a method to eliminate unneccesary content in the final image. With multi-stage builds, you use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image. This ensures that the final build is compact and least complex. You can read more about this [here](https://docs.docker.com/develop/develop-images/multistage-build/).
	3. Non-root User
		 The USER instruction in a Dockerfile specifies that the default user
identity for running containers based on this image isn’t root. If
you don’t want all your containers running as root, specify a non root user in all your Dockerfiles.
	4. RUN commands
		A Dockerfile RUN command can be used to run any arbitrary command. If a dockerfile is compromised, an attacker can run any code using this command. Ensure that access to dockerfile is limited and close attention is paid to code reviewing any changes made.
	5. Volume Mounts
		Using the -v option, you can mount a host directory into a container so that it is available from the container. And there is nothing to stop you from mounting the host’s root directory into a container. An attacker who compromises this container is root on the host, with full access to the entire host filesystem. Some example include		
		- Mounting /etc would permit modifying the host’s /etc/passwd
file from within the container, or messing with cron jobs, or
init, or systemd.
		- Mounting /bin or similar directories such as /usr/bin or
/usr/sbin would allow the container to write executables into
the host directory—including overwriting existing
executables.
		- Mounting host log directories into a container could enable
an attacker to modify the logs to erase traces of their
dastardly deeds on that host.
	6. Secrets in Dockerfile
		Avoid using credentials, passwords or any secrets in the Dokcerfile. One safe way is to pass the secret during buildtime. Buildkit has `--secret` flag that can be used. Refer [this](https://www.gasparevitta.com/posts/advanced-docker-secrets-buildkit/) blog for an example of how this can be implemented. Another method is to use the [docker secrets](https://docs.docker.com/engine/swarm/secrets/) , but this requries the docker to be in [swarm](https://docs.docker.com/engine/swarm/) mode.
	7. Avoid setuid binaries
		Setuid binaries can lead to privilege escalation and it always a good idea to avoid these.
		
	## Storing and deploying images
	Once images are build these needs to be stored in a registry. If any attacker can replace or modify container, it can lead to arbitrary code execution therefore it is important to controll access to these registries.
		1. Running own registry
		Many organizations run their own registries or use a managed registry like [Docker Hub](https://www.docker.com/pricing). In both cases, access must be audited and controlled.
		2. Signing images
		Image signing is where an organization can sign their image before they push it to the container, so that the customer can use it safely. Signing is a way to prove that the image has not been tampered with. If someone make changes to the image, the signature also changes thereby invalidating the image. [Docker content trust](https://docs.docker.com/engine/security/trust/) provides a way to use digital signatures to verify the integrity of an image.


## References
- https://www.freecodecamp.org/news/demystifying-containers-101-a-deep-dive-into-container-technology-for-beginners-d7b60d8511c1/
- https://gtfobins.github.io/gtfobins/docker/
- https://github.com/moby/buildkit
- https://podman.io/
- https://buildah.io/
- https://github.com/bazelbuild/rules_docker
- https://github.com/GoogleContainerTools/kaniko
- https://github.com/genuinetools/img
- https://github.com/GoogleContainerTools/distroless
- https://docs.docker.com/develop/develop-images/multistage-build/
- https://www.gasparevitta.com/posts/advanced-docker-secrets-buildkit/
- https://docs.docker.com/engine/swarm/secrets/
- https://docs.docker.com/engine/swarm/
- https://docs.docker.com/engine/security/trust/

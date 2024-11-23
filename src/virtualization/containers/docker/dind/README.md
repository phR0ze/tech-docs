# Docker in Docker

### Quick links
* [CI Setup](#ci-setup)
  * [Build custom Dockerfile](#build-custom-dockerfile)
  * [Launch CI daemon](#launch-ci-daemon)
  * [Configure registry access](#configure-registry-access)
* [Socket mount solution](#socket-mount-solution)

## CI Setup
Docker in Docker involves running Docker within a Docker container. Instead of interacting with the 
host's Docker daemon, a new Docker engine is spawned within a container.

**NOTE** For the purposes of my research below I'm building this out to have access to Google 
Platform Resources using Github as my project store. This means that there will be custom binaries 
and execution steps for this setup.

### Build custom Dockerfile
I'm starting from the official [Docker image](https://hub.docker.com/_/docker) using the `dind` tag 
which is an `alpine` based image built on the `cli` image which contains both `docker` and 
`docker-compose` binaries.

```Dockerfile
# Official Docker in Docker base to build on
FROM docker:27.3.1-dind-alpine3.20 AS docker-dind

# Include gcloud sdk support
# https://github.com/GoogleCloudPlatform/cloud-sdk-docker/blob/master/alpine/Dockerfile
ENV CLOUD_SDK_VERSION="496.0.0"
ENV PATH=/google-cloud-sdk/bin:$PATH
RUN if [ `uname -m` = 'x86_64' ]; then echo -n "x86_64" > /tmp/arch; else echo -n "arm" > /tmp/arch; fi;
RUN ARCH=`cat /tmp/arch` && apk --no-cache upgrade && apk --no-cache add \
  curl python3 py3-crcmod py3-openssl bash libc6-compat openssh-client git gnupg && \
    curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-${CLOUD_SDK_VERSION}-linux-${ARCH}.tar.gz && \
    tar xzf google-cloud-cli-${CLOUD_SDK_VERSION}-linux-${ARCH}.tar.gz && \
    rm google-cloud-cli-${CLOUD_SDK_VERSION}-linux-${ARCH}.tar.gz && \
    gcloud config set core/disable_usage_reporting true && \
    gcloud config set component_manager/disable_update_check true && \
    gcloud config set metrics/environment docker_image_alpine && \
    gcloud --version
RUN git config --system credential.'https://source.developers.google.com'.helper gcloud.sh
VOLUME ["/root/.config"]

# CI Configuration
COPY configs/* /root/
WORKDIR /workspace

# Clone the target project with the given secret token
RUN --mount=type=secret,id=GITHUB_TOKEN,env=GITHUB_TOKEN \
  git clone https://oauth2:$GITHUB_TOKEN@github.com/<owner>/<repo> /workspace
```

Build: `docker build --secret id=GITHUB_TOKEN -t ci:latest .`

### Launch CI daemon
Note: pass in environemnt variables like `GITHUB_TOKEN` with the `-e` flag for use at runtime

1. Start the container as a daemon
   ```bash
   $ docker run --rm --privileged -e GITHUB_TOKEN -d --name ci ci:latest
   ```
2. Shell in to work directly in the container for temporary hacks
   ```bash
   $ docker exec -it ci /bin/bash

   # Install tooling
   $ apk add vim

   # Configure github access to work around dependency syntax
   $ cat <<EOF > ~/.gitconfig 
   [url "https://oauth2:$GITHUB_TOKEN@github.com/<company>"]
     insteadOf = git@github.com:<company>
   ```
3. Kill the container once your happy
   ```bash
   $ docker container kill ci
   ```

### Configure registry access
1. Login to gcloud and follow steps in browser copy the code to terminal
   ```bash
   $ docker exec -it ci gcloud auth login --update-adc
   ```
2. Setup docker access to google registry
   ```bash
   $ docker exec -it ci gcloud auth configure-docker <company-docker.pkg.dev>
   ```

## Socket mount solution
By mounting the Docker daemon's Unix socket `/var/run/docker.sock` from the host into the container 
we can use it to manage Docker. This opens up the possibility of building, pushing or pulling 
images from the configured registry and/or any other docker operations. The actual operations 
actually happen on the host machine that is running your base docker container.

Note: [Jérôme Petazzoni](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/), 
the creator of the dind implementation recommends adopting the socket-based approach as the preferred 
solution. The reasons he notes are that bind mounting your host's daemon socket is safer, more 
flexible, and just as feature-complete as starting a full dind container.

### Docker cli vs dind
Docker has official images available [from docker hub](https://hub.docker.com/_/docker) customized 
for each task. The `cli` is an alpine based image with `ca-certificates`, `openssh-client` and `git` 
dependencies installed as well as `docker`, `buildx` and `compose`. The `dind` version builds on top 
of the cli version adding a number of networking packages and additional configuration.

### OSX Docker Socket Solution
> Note: I tried this but ran into immediate errors and switched over to [CI setup](#ci-setup) 
> instead.

I'll be using the official [Docker image](https://hub.docker.com/_/docker)

1. Change the docker sock permissions
   ```bash
   $ sudo chmod 666 /var/run/docker.sock
   ```
2. Start the Docker container in interactive mode mounting the `docker.sock` as a volume.
   ```bash
   $ docker run --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -ti docker
   ```
3. Once inside the container execute commands as desired
   ```bash
   $ docker pull ubuntu 
   ```

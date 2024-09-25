# Docker

## Docker in Docker
By mounting the Docker daemon's Unix socket `/var/run/docker.sock` from the host into the container 
we can use it to manage Docker. This opens up the possibility of building, pushing or pulling 
images from the configured registry and any other docker operations. The actual operations actually 
happen on the host machine that is running your base docker container.

Note: According to Jérôme Petazzoni, the creator of the dind implementation, adopting the 
socket-based approach should be your preferred solution. Bind mounting your host's daemon socket is 
safer, more flexible, and just as feature-complete as starting a dind container.

### Docker cli vs dind
Docker has official images available [from docker hub](https://hub.docker.com/_/docker) customized 
for each task. The `cli` is an alpine based image with `ca-certificates`, `openssh-client` and `git` 
dependencies installed as well as `docker`, `buildx` and `compose`. The `dind` version builds on top 
of the cli version adding a number of networking packages and additional configuration.




Example using official [Docker image](https://hub.docker.com/_/docker)
1. Change the docker sock permissions
   ```bash
   $ sudo chmod 666 /var/run/docker.sock
   ```
2. Start the Docker container in interactive mode mounting the `docker.sock` as a volume.
   ```bash
   $ docker run --privileged -v /var/run/docker.sock:/var/run/docker.sock -ti docker
   ```
3. Once inside the container execute commands as desired
   ```bash
   $ docker pull ubuntu 
   ```

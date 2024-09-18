# Docker

## Docker in Docker
By mounting the Docker daemon's Unix socket `/var/run/docker.sock` from the host into the container 
we can use it to manage Docker. This opens up the possibility of building and pushing or pulling 
images from the configured registry. The actual operations actually happen on the host machine that 
is running your base docker container.

Note: According to Jérôme Petazzoni, the creator of the dind implementation, adopting the 
socket-based approach should be your preferred solution. Bind mounting your host's daemon socket is 
safer, more flexible, and just as feature-complete as starting a dind container.

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

# Portainer <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Portainer is a lightweight management UI which allows you to easily manage your Docker host. It
consists of a single container that can run on any Docker engine including `Docker for Synology DSM`.

### Quick links
* [.. up dir](..)
* [Installing](#installing)
  * [Install Container Manager](#install-container-manager)
  * [Install Portainer](#install-portainer)
* [Updating](#updating)
* [Deploy stack](#deploy-stack)

## Installing
Before you can install Portainer you must first install Container Manager i.e. Docker to run
Portainer. Container Manager is slower and more cumbersome to use though so we'll continue on to
install Portainer on top of it.

### Install Container Manager
First step is to install Docker a.k.a `Container Manager` in the Synology world.

**References**
* [Synology help](https://www.synology.com/en-us/dsm/feature/docker)

1. Launch the `Package Center`
2. Search for `Container Manager` and click `Install`
3. Click `Next` on the `Configure bridge network`
4. Click `Done` to complete the install and start it

### Install Portainer
1. Download the portainer compose file
   1. Navigate to the official [Portainer instructions](https://docs.portainer.io/start/install-ce/server/docker/linux#docker-compose)
   2. Copy the `Docker Compose` file and save locally as `docker-compose.yml`
   3. Change the data volume to `./data:/data` and drop the corresponding volume section
      * Note: this will be relative to `Set Path` we'll choose later down
   4. Drop the `8000:8000` port
      ```yaml
      services:
        portainer:
          container_name: portainer
          image: portainer/portainer-ce:lts
          restart: always
          ports:
            - 9443:9443
          volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - ./data:/data
      networks:
        default:
          name: portainer_network
      ```

2. Create a folder for portainer
   1. Launch and navigate to `File Station >docker`
   2. Create a new folder called `portainer` all lower case
   3. And inside that create another folder called `data`

3. Create the Container Manager portainer project
   1. Launch and navigate to `Container Manager >Project`
   2. Click `Create`
   3. Set the `Project name` to `portainer` all lower case
   4. For the `Path` click `Set Path` and select the path we just created `/docker/portainer`
   5. For `Source` keep `Upload docker-compose.yml`
   6. For `File` click the `Browse` button to the right and find `docker-compose.yml` from earlier
   7. Click the `Next` to advance
   8. One the `Web portal settings` just skip by clicking `Next`
   9. Finally click `Done`

4. Configure Portainer first run
   1. Navigate to `https` your Synology IP address port `9443`
   2. Fill out username and password choice
   3. Change the environment instances name as desired e.g. `synology`

## Updating
Container Manager may need to be started an let sit for a while before it recognizes there is an
update.

1. Launch `Container Manager`
2. Switch to the `Image` section
3. Find your target app's container and click `Update`
4. After the image is updated switch the `Container` section
5. Either start or restart Portainer

## Deploy stack
Portainer uses the term `stack` to refer to a docker or docker compose based application to be
deployed.



# Ente <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Ente claims to be the best place to preserve your memories. It is fully open source, self-hosted with 
end-to-end encryption. Howeve they do have a paid option for storing your media in the cloud.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)

## Overview
Ente Photos is an alternative to Apple or Google Photos. The server and client code is all available 
in a mono repo on github [ente-io/ente](https://github.com/ente-io/ente) as they [open sourced the 
server code in Mar. 1st of 2024](https://ente.io/blog/open-sourcing-our-server/). This is the same
code base they use for their paid services so it is a live, breathing, continuously updated 
repository were they actively develop.

**References**
* [Ente site](https://ente.io/)

## Review
**Summary**
* End-to-end encryption increases risk of locking family photos making them inaccessible and is not 
needed for this use case.
* Deployability was tedious and fraught with errors
* With a focus on a paid model there isn't much support for shared family space

**Notes**
* Longevity
  - Ente is backed with a paid service so there is constant active development
  - Ente is fully open source so it could be forked and maintained by the community
  - Using GoLang on the backend and Flutter on the front end
* Community
  - Ente is constantly being worked on as a paid product
  - Only being open-sourced a year ago it doesn't have much following in the self-hosted community
* Deployability
  - No portainer integration out of the box :(
  - Main branch failed to build, had to go back to `v0.9.53` to get something working
  - Required a lot of sleuthing, manual file changes and cli tweaks to get up and running
  - Only sparse minimal NixOS support
  - Documentation was lacking
* General
  - Uses end-to-end encryption with MinIO for encrypted file object storage. Although interesting 
    this isn't a great fit for family use where preserving file access and concern of loosing 
    encryption access may be an issue.
* Web UI
  - Geared for paid users with free plan branding and complicated cli steps to remove
  - Plays gifs, but not videos unless clicked
  - Modern escape to exit views support
  - Supports a hidden space
* Mobile
  - TBD

### Self-hosted installation
Ente provides a [self-hosted install document](https://help.ente.io/self-hosting/) to get you 
started with Docker compose configuration. 

**References**
* [Ente selfhosted tutorial](https://github.com/ente-io/ente/discussions/3778)

**Dependencies**
* Postgres
* MinIO - WebUI available at http://127.0.0.1:3201
* MC
* Socat

1. Clone the repo and update submodules
   ```bash
   $ git clone https://github.com/ente-io/ente
   $ cd ente
   $ git submodule update --init --recursive
   ```
2. Build the server image
   ```bash
   $ cd server
   $ docker build --no-cache --progress=plain -t ente-server:latest -f Dockerfile .
   ```
3. Build the web client image
   1. Switch to the web dir
      ```bash
      $ cd ../web
      ```
   2. Create the web Dockerfile
      ```Dockerfile
      FROM node:21-alpine AS builder
      
      WORKDIR /app
      COPY . .
      
      ARG NEXT_PUBLIC_ENTE_ENDPOINT=https://ente.example.com
      ENV NEXT_PUBLIC_ENTE_ENDPOINT=${NEXT_PUBLIC_ENTE_ENDPOINT}
      
      ARG NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT=https://album.example.com
      ENV NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT=${NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT}
      
      ARG NEXT_PUBLIC_ENTE_ACCOUNTS_URL=https://accounts.example.com
      ENV NEXT_PUBLIC_ENTE_ACCOUNTS_URL=${NEXT_PUBLIC_ENTE_ACCOUNTS_URL}
      
      RUN yarn install && yarn build:photos && yarn build:auth && yarn build:accounts
      
      FROM node:21-alpine
      
      WORKDIR /app
      COPY --from=builder /app/apps/photos/out /app/photos
      COPY --from=builder /app/apps/auth/out /app/auth
      COPY --from=builder /app/apps/accounts/out /app/accounts
      
      RUN npm install -g serve
      
      ENV WEBPORT=3000
      EXPOSE ${WEBPORT}
      
      ENV PUBLICPORT=3002
      EXPOSE ${PUBLICPORT}
      
      ENV ACCOUNTS=3001
      EXPOSE ${ACCOUNTS}
      
      ENV AUTHPORT=3005
      EXPOSE ${AUTHPORT}
      
      CMD ["sh", "-c", "serve /app/photos -l tcp://0.0.0.0:${WEBPORT} -l tcp://0.0.0.0:${PUBLICPORT} & serve /app/auth -l tcp://0.0.0.0:${AUTHPORT} & serve /app/accounts -l tcp://0.0.0.0:${ACCOUNTS}"]
      ```
   3. Build the client
      ```bash
      $ docker build --no-cache --progress=plain -t ente-web:latest -f Dockerfile .
      ```
4. Switch back the root and create configuration files
   1. Create the `credentials.yaml`
      ```yaml
      db:
          host: postgres
          port: 5432
          name: ente_db
          user: pguser
          password: changemechangeme
      
      s3:
          are_local_buckets: true
          b2-eu-cen:
              key: admin
              secret: changemechangeme
              endpoint: localhost:3200
              region: eu-central-2
              bucket: b2-eu-cen
          wasabi-eu-central-2-v3:
              key: admin
              secret: changemechangeme
              endpoint: localhost:3200
              region: eu-central-2
              bucket: wasabi-eu-central-2-v3
              compliance: false
          scw-eu-fr-v3:
              key: admin
              secret: changemechangeme
              endpoint: localhost:3200
              region: eu-central-2
              bucket: scw-eu-fr-v3
      ```
   2. Create the `minio-provision.yaml`
      ```yaml
      #!/bin/sh
      while ! mc config host add h0 http://minio:3200 admin changemechangeme
      do
         echo "waiting for minio..."
         sleep 0.5
      done
      
      cd /data
      mc mb -p b2-eu-cen
      mc mb -p wasabi-eu-central-2-v3
      mc mb -p scw-eu-fr-v3
      ```
   3. Create the `museum.yaml`
      ```yaml
      http:
          # use-tls: true
      apps:
          public-albums: https://pics.example.com
      webauthn:
          rpid: accounts.example.com
          rporigins:
              - "https://accounts.example.com"
      s3:
          are_local_buckets: true
          b2-eu-cen:
              key: admin
              secret: changemechangeme
              endpoint: localhost:3200
              region: eu-central-2
              bucket: b2-eu-cen
      ```
   4. Create the `compose.yaml`
      ```yaml
      services:
        museum:
          image: ente-server:latest
          container_name: ente-museum
          restart: always
          ports:
            - 8080:8080 # API
          depends_on:
            postgres:
              condition: service_healthy
          environment:
            # Pass-in the config to connect to the DB and MinIO
            ENTE_CREDENTIALS_FILE: /credentials.yaml
          volumes:
            - ./custom-logs:/var/logs
            - ./museum.yaml:/museum.yaml:ro
            - ./credentials.yaml:/credentials.yaml:ro
            - ./data:/data:ro
          networks:
            - ente
        web:
          image: ente-web:latest
          container_name: ente-web
          restart: always
          environment:
            NEXT_PUBLIC_ENTE_ENDPOINT: https://ente.example.com
            NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT: https://album.example.com
            NEXT_PUBLIC_ENTE_ACCOUNTS_URL: https://accounts.example.com
          ports:
            - "3000:3000"
            - "3001:3001"
            - "3002:3002"
            - "3005:3005"
          networks:
            - ente
        socat:
          image: alpine/socat
          container_name: ente-socat
          restart: always
          network_mode: service:museum
          depends_on:
            - museum
          command: "TCP-LISTEN:3200,fork,reuseaddr TCP:minio:3200"
        postgres:
          image: postgres:16
          container_name: ente-postgres
          restart: always
          ports:
            - 5432:5432
          environment:
            POSTGRES_USER: pguser
            POSTGRES_PASSWORD: changemechangeme
            POSTGRES_DB: ente_db
          # Wait for postgres to be accept connections before starting museum.
          healthcheck:
            test:
              [
                "CMD",
                "pg_isready",
                "-q",
                "-d",
                "ente_db",
                "-U",
                "pguser"
              ]
            start_period: 40s
            start_interval: 1s
          volumes:
            - ./postgres-data:/var/lib/postgresql/data
          networks:
            - ente
        minio:
          image: minio/minio
          container_name: ente-minio
          restart: always
          ports:
            - 3200:3200 # API
            - 3201:3201 # Console
          environment:
            MINIO_ROOT_USER: admin
            MINIO_ROOT_PASSWORD: changemechangeme
          command: server /data --address ":3200" --console-address ":3201"
          volumes:
            - ./minio-data:/data
          networks:
            - ente
        minio-provision:
          image: minio/mc
          container_name: ente-provision
          depends_on:
            - minio
          volumes:
            - ./minio-provision.sh:/provision.sh:ro
            - ./minio-data:/data
          networks:
            - ente
          entrypoint: sh /provision.sh
      networks:
        ente:
          external: true
      ```
5. Create the network and run it
   1. Create network
      ```bash
      $ podman network create ente
      ```
   2. Run it
      ```bash
      $ docker compose up -d
      ```
   3. I tried to create my account and it failed had to switch to `http://localhost:8080` for server
   4. Look at the server logs for the verification code `docker logs -f ente-museum`

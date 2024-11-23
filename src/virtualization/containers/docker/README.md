# Docker

## Build Secrets
A build secret is any piece of sensitive information, such as a password or API token, consumed as 
part of your application's build process.

**References**
* [Docker docs - Build secrets](https://docs.docker.com/build/building/secrets/)

Build arguments and environment variables are inappropriate for passing secrets to your build, 
because they persist in the final image. Instead, you should use secret mounts or SSH mounts, which 
expose secrets to your builds securely.

### Secret mounts
To pass a secret to a build use the `docker build --secret` flag. To then consume the secret in the 
Dockerfile and make it accessible to the `RUN` instruction use the `--mount=type=secret`.

The source of the secret can be either a file or an environment variable. The type will be detected 
automatically.

**Build Example:**
```bash
$ docker build --secret GITHUB_TOKEN .
```

**Dockerfile Example:**
```Dockerfile
RUN --mount=type=secret,id=GITHUB_TOKEN,env=GITHUB_TOKEN \
  git clone https://oauth2:$GITHUB_TOKEN@github.com...
```

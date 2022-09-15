---
title: Setting up (or transitioning to) Docker Compose v.2 on Linux
post_excerpt: Upgrading to Docker Compose V2 from legacy v1 without Docker Desktop
taxonomy:
    category:
        - knowledge
        - notes
---

### Installing Docker, without Docker Desktop?

Straight from [the documentation](https://github.com/docker/compose/tree/v2#linux):
> Docker Compose is a tool for running multi-container applications on Docker defined using the Compose file format. A Compose file is used to define how the one or more containers that make up your application are configured. 

Version 2 of Docker Compose also uses a different syntax (`docker compose ...` rather than `docker-compose ...`), and the installation process for v2 also differs from that of v1. This guide is intended for developers wanting to install Docker Compose v2 and a quick start guide on using it effectively.

You will get Docker Compose by installing Docker Desktop and this is most likely how many Linux users would go about it. However, on some distributions, including Ubuntu, Docker Desktop will require user to have the latest LTS (long term support) version of its distro -- no small feat when this isn't always an option, such as in the case of a production machine. 

Pulling [from the documentation](https://docs.docker.com/desktop/install/ubuntu/):
> To install Docker Desktop successfully on Ubuntu, you must:
> 
> - Meet the system requirements
> - Have a 64-bit version of either Ubuntu Jammy Jellyfish 22.04 (LTS) or Ubuntu Impish Indri 21.10. Docker Desktop is supported on x86_64 (or amd64) archi

On more than one of my machines, I'm using an older LTS -- which means I'm out of luck with Docker Desktop. This is the motivation of writing this guide on installing Docker and Docker Compose on Linux without the Docker Desktop bundle.

### Installing Docker and Docker Compose on Linux

Docker Compose requires the use of Docker so a quick installation guide will be handy reference here. 

First, set up Docker's repository and install them with `cURL` is the recommended approach and this is done with the following lines of code:

```bash
$ sudo apt update
$ sudo apt install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Add Docker’s official GPG key and set up the repository

```bash
$ sudo mkdir -p /etc/apt/keyrings

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Use the following command to set up the repository:

$ echo \
 "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Finally, install Docker Engine and Docker Compose:

```bash
$ sudo apt update
$ sudo apt install docker-ce docker-ce-cli docker-compose-plugin
```

Verify the installation:
```bash
$ docker version

Client: Docker Engine - Community
 Version:           20.10.17
 API version:       1.41
 Go version:        go1.17.11
 Git commit:        100c701
 Built:             Mon Jun  6 23:02:57 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.17
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.17.11
  Git commit:       a89b842
  Built:            Mon Jun  6 23:01:03 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.6
  GitCommit:        10c12954828e7c7c9b6e0ea9b0c02b01407d3ae1
 runc:
  Version:          1.1.2
  GitCommit:        v1.1.2-0-ga916309
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0


```

### Installing Docker Compose from binaries

If you are restricted from updating to the latest versions of your Linux distro to meet the requirement of Docker Desktop, and would like to install from the provided binaries, read on!

1. Hop over to the [release page](https://github.com/docker/compose/releases) and find the relevant binary for your OS. Firing up your terminal and using the `uname` command will help:

    ```bash
    $ echo "$(uname -s) $(uname -m)"
    Linux x86_64
    ```

    Replace the following command with the latest version from the aforementioned release page and the appropriate kernel name. The following command uses `cUrl` to download that and store it to `/usr/local/lib/docker/cli-plugins`, making it available system-wide:

    ```bash
    $ sudo curl -L sudo curl -L "https://github.com/docker/compose/releases/download/2.11.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/lib/docker/cli-plugins
    ```

    Rename it to `docker-compose` using the `mv` command:
    ```bash
    $ mv docker-compose-linux-x86_64 docker-compose
    ```

    Skip to (3)!

2. (Alternative) You can also download the right binary for your distribution manually, rename it to `docker-compose` post-download and then copy it to `/usr/local/lib/docker/cli-plugins` using the `cp` or `mv` (for 'move') command. 
    - Create that directory if it doesn't yet exist before copying it over

3. Make the file executable with `chmod +x`:

    ```bash
    $ sudo chmod +x /home/your_username/.docker/cli-plugins/docker-compose
    ```

4. Verify that the installation was successful:

    ```bash
    $ docker compose version
    Docker Compose version v2.11.0
    ```

#### Transitioning away from `docker-compose` (version 1)

Docker Compose V2 is a major version bump release of Docker Compose. It has been completely rewritten from scratch in Golang (V1 was in Python).

If you already have a pre-existing version of `docker-compose` (version 1), the installation of version 2 doesn't remove that legacy version:

```bash
$ docker-compose --version
docker-compose version 1.28.2, build 67630359
```

If you want to transition from that legacy docker-compose 1.xx to Compose V2, consider [installing compose-switch](https://github.com/docker/compose-switch) to translate `docker-compose ...` commands into Compose V2's `docker compose ...`. Post-transition, you can remove the legacy version and use Compose V2 exclusively.

If you have a local Compose file, you should now run `docker compose up` (instead of `docker-compose up`). For completeness sake, an example Compose file looks like the following:

```yaml
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
  redis:
    image: redis
```

### Further learning resources on Docker
If you make it this far, a good "next step" is Stane's write-up on Docker and the official documentation:
    
- [The Essential Guide to Docker](https://supertype.ai/notes/docker-guide/) 
- [Docker's Get Started Guide](https://docs.docker.com/get-started/)

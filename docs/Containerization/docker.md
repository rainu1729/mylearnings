Some of the useful commands in Docker

**Core Concepts Covered:**

1.  **Docker Installation:** Install docker on Ubuntu.

2.  **Basic Commands:** finding, pulling, pushing, and cleaning up images.

---

### Essential Daily Docker Commands



**1. Docker Installation**

* Add Docker's official GPG key:

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
* Add the repository to Apt sources. 

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

* Install the latest version of docker from repository.

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

* Test run

```
sudo docker run hello-world
```

* Add the logged in user to Docker unix group so that current user can run commands without sudo

```
sudo usermod -aG docker $USER
newgrp docker
#Test the command
docker run hello-world
```
**2. Basic Commands**

* Pull alpine linux image latest version (tag) and open container terminal

```
docker run -it alpine --name MyAlphineContainer
```

* List all available images locally

```
docker images
```

* Login to running container username:rainun containerName:alpine-container-1
```
docker exec -it -u rainun alpine-container-1 /bin/bash
```

* Forcefully remove image get the image id from the command docker images
```
docker rmi -f image_id
```

* Docker list running container
```
docker ps 
docker container ls
docker container ps 
```
* Docker list all container including inactive ones
```
docker ps -a
docker container ls -a
docker container ps -a

# Display specific columns of the container including size
docker container ls -a -s --format 'table{{.Names}}\t{{.Size}}\t{{.Status}}'

#Apply filter while displaying containers

#Possible value of docker container status 
#created, running, paused, restarting, removing, exited, dead
docker container ls -f "status=exited"
docker container ls --filter "status=exited"

```
* Run start a exited container 
```
docker start <<ContainerName/ContainerId>>
```


# NGINX in VPS with DOCKER SWARM

> Oficial Documentation: [Docker Swarm](https://docs.docker.com/engine/swarm/)

Docker Swarm is a native container orchestrator from docker.

Docker Swarm use a concept calls: entry  routing mode ( or ingress routing mesh).

When you define the port in your docker-compose.yml (like this- ports: -"80:80"), docker swarm make this:


* 1 -It configures a intern load balancer that listening on port 80 in all nodes from your Swarm(manager and workers).


* 2 - When a request arrives in aby node in the port 80, the load balancer forwards to any one replics from Nginx that are runing in your overlay network.  

>  Swarm make a inteligent layer, distributing requests to port 80 and any replic avilable in your service, ensuring resilience and high availability.


### Start Docker Swarm 

With Docker and Docker Compose installed, make this:

```
 docker swarm init --advertise-addr $MY_IP
```
>  Your machine's IP can be get with command: ifconfig or ip addr.

### Adding : "uniting" the worker at the cluster 

The option "uniting a worker" is the way like Docker transform a only one server in a distributed cluster, which the principal proposit from orchestration. 

To add worker at the cluster: In the terminal from worker, paste generated command in the manager to connect at the cluster:


```
docker swarm join --token $TOKEN $MY_IP
```

### Create a Network between Nginx and Apps 

We need create a overlay network that will be the "backbone" of comunication between all our services. 
You just need make this only one time in your lead swarm node.

```

docker network create --attachable --driver overlay my-project-network-name
```
### Starting NGINX 

In this repository there is a example from docker-compose.yml and a directory structury and files that be can use:

- my-project
    - nginx
        - sites
        - nginx.conf
    - docker-compose.yml



To start Nginx, browser to project's root folder and execute: 


```
docker stack deploy -c docker-compose.yml nginx-stack
```

### Reload Nginx while keeping online services 

> After includes a new site (site*.conf) or configuration update fromsite, use:


```
docker stack deploy -c docker-compose.yml nginx-stack
```

### Configure your projects with Docker Swarm 

```
docker stack deploy -c docker-compose.yml site1-stack
```
---

> Docker Compose from Site 1:

**docker-compose.yml**

```
version: '3.8'

services:
  site1-app:
    build: .
    image: my-site1-image
    networks:
      - my-project-network-name
    deploy:
      mode: replicated
      replicas: 3
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
      restart_policy:
        condition: on-failure

networks:
  my-project-network-name:
    external: true
```
---

> Docker Compose from Site 2:

**docker-compose.yml**

```
version: '3.8'

services:
  site1-app:
    build: .
    image: my-site2-image
    networks:
      - my-project-network-name
    deploy:
      mode: replicated
      replicas: 3
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
      restart_policy:
        condition: on-failure

networks:
  my-project-network-name:
    external: true
```

```
docker stack deploy -c docker-compose.yml site2-stack
```


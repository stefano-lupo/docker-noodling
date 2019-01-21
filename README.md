# Docker
## Containers
- Create `Dockerfile` which lets us define things the image needs
  - Can run commands to init the container
  - Can define command to run once container set uo
- Building the image
  - `docker build -t test_docker_app .`
- Running the container
  - Map real port 4000 to virtual port 80
  - `docker run -p 4000:80 friendlyhello` 
- Tag the docker image
  - `docker tag image stefanolupo/<image_name>:<version(e.g. latest)>`
  - `docker image ls`
- Push to docker hub
  - `docker push stefanolupo/<image_name>:tag`
- Run anywhere (auto pulls the container)
  - `docker run -p 4000:80 stefanolupo/<image_name>:tag`


## Services
Used for deploying multiple instances of image on a single machine
### Create docker-compose.yml
Creates 5 instances of the containers and limits their resources. Also hooks up the port forwards.
Create a default network `webnet` which handles load balancing across the instances
```yaml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

### Initialize as a swarm
```bash
docker swarm init
```

### Deploy Service
```bash
docker stack deploy -c docker-compose.yml <service_name>
```

### Have a look at service
- List services `docker service ls`
- List processes (tasks) in service: `docker service ps <service_name>`

### Scale service manually
- Just adjust # replicas and rerun deploy

### Quit
- Take down app: `docker stack rm <service_name>`
- Take down swarm: `docker swarm leave --force`

## Swarms
- Uses docker-machine to manage machines in cluster
- Swarms are group of focker machines joined into a cluster
- Docker commands are now executed cluster wide by a swarm manager
- Swarm managers can use strategies for assigning containers to nodes
  - Global: all get one
  - Emptiest node: prioritize those with least work currently
- Swarm managers are the only nodes who can orchestrate cluster
- **Note: Docker swarm management port is 2377 - always use this (just leave default)**
- **Note: Docker daemon runs on port 2376 - awlays use this (just leave default)**
### Creating nodes in cluster
Can use docker itself to provision nodes for a bunch of aaS providers. 
#### Google Cloud
- **Note: Probably need to set application default credentials**
  - `gcloud auth application-default login`
- This handles installing docker, creating SSH keys and opening firewall ports automatically
```bash
# Add docker machine of an existing instance
docker-machine create \
--driver google \
--google-project ndn-thesis
--google-address <ip_add> \
--google-zone europe-west4-a
--google-use-existing
<google_machine_name>

# Add docker machine from new instance
docker-machine create \
--driver google \
--google-project ndn-thesis \
--google-machine-type f1-micro \ # Chepaest 
--google-zone europe-west4-a
<machine_name>
```
- Now see the machines we have `docker-machine ls`
  - Might need to extend timeout `docker-machine ls -t 20`

### Setting up the swarm
- IPs come from `docker-machine ls`
```bash
# Make <mach_name> the swarm leader (not sure about auto choosing IP)
docker-machine ssh <mach_name> "docker swarm init --advertise-addr <ip_to_use>"

# Get other machines to join the swarm
docker-machine ssh <other_mach_name> "docker swarm join --token <token> <ip>
```
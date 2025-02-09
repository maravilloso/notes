Docker
=====

# Images
```bash
# See Locally installed images
docker images

# Build an image from the Dockerfile in the same folder
docker build -t qvbvntv-maria106-pdi71 .

# See the complete sequence of commands with which an image was built
docker history --format "{{.ID}}: {{.CreatedBy}}" --no-trunc artifactory.qvantel.net/migration-tool

# Delete an image
docker rmi <image-name>
```  
___
# Pushing images to artifactory
  Note you first need to `docker login` using your Artifactory **User Profile** name, and a generated API Key as password:

```bash
  docker login artifactory.qvantel.net
  
  # Now the image needs to be tagged i.e.:
  docker tag datmig-dbclients artifactory.qvantel.net/datmig-dbclient
  
  # Finally:
  docker push artifactory.qvantel.net/datmig-dbclient
```
___
# Running containers
```bash
  # Run an image into a container
  docker run --rm -it prueba1
  
  # Passing environment parameter
  docker run --rm -it --env MARIADB_ROOT_PASSWORD=12345 qvbvntv-maria106-pdi71
  
  # Map local folder to a container folder
  docker run -it -v /tmp:/tmp --rm --net host artifactory.qvantel.net/docker-cassandra-tools:3.11.11.9_master_a50beba bash
  
  # See running containers
  docker container list
  # List even those stopped
  docker container ls --all
  
  # Remove all stopped containers
  docker rm $(docker ps --filter status=exited -q)
  
  # Get the IP of a running container
  docker inspect -f "{{ .NetworkSettings.IPAddress }}" 99eb9db26bc9
  
  # Connect to a running container as a specific user and open a bash session
  docker exec --user migration -it 1565e370010f bash
  
  # Copy a file to/from a running container
  docker cp pg_counts.sql f46cc55cfe8a:pg_counts.sql
  docker cp f46cc55cfe8a:out.csv batch_starting_tool.csv
```
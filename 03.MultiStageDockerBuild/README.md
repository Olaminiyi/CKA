## Multi Stage Docker Build

Multi stage Docker build is used when we want to reduce the size of the image or improve the performance of container

-  Clone a sample git repo

```
git clone https://github.com/piyushsachdeva/todoapp-docker.git
```
- cd into the directory

```
cd todoapp-docker/
```

- Create an empty file with the name Dockerfile
```
touch Dockerfile
```

- Using the text editor of your choice, paste the below content. I'm using vi
```
vi Dockerfile
```
- inside the the dockerfile
```
FROM node:18-alpine AS installer
WORKDIR /app
COPY package*.json ./
RUN npm install 
COPY . .
RUN npm run build
FROM nginx:latest AS deployer
COPY --from=installer /app/build /usr/share/nginx/html
```

- We are using node:18-alpine as an installer, that's the first stage and nginx as the deployer, the second stage

- Build the docker image using the application code and Dockerfile
```
docker build -t todoapp-docker .
```

- Verify the image has been created and stored locally using the below command:
```
docker images
```

- Create a public repository on hub.docker.com and push the image to remote repo

```
docker login
docker tag todoapp-docker:latest username/new-reponame:tagname
docker images
docker push username/new-reponame:tagname
```

- To pull the image to another environment, you can use the below command
```
docker pull username/new-reponame:tagname
```

- To start the docker container, use the below command
```
docker run -dp 3000:80 username/new-reponame:tagname
```

- To enter(exec) into the container, use the below command
```
docker exec -it containername sh
or
docker exec -it containerid sh
```

- To view docker logs
```
docker logs containername
or
docker logs containerid
```

- To view the content of Docker container
```
docker inspect
```
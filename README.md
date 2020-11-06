# How to use docker.

### Setup vagrant
```
vagrant up
```
### ssh into vagrant VM
```
vagrant ssh
```
### Run below steps in vagrant VM
```
# Pull docker image
docker pull nginx
# Run the image in a container
docker run --rm -p 80:80 --name nginx-for-workshop nginx
```
### Test it in vagrant VM
```
curl localhost
```
### Try to edit home page

* Edit index.html

```
touch index.html
tee -a index.html << END
<html>
  <header>
    <title>Workshop</title>
  </header>
  <body>Awesome Workshop</body>
</html>
END
```

* Create volume between host and docker container

```
docker run --rm -p 80:80 -v "$PWD":/usr/share/nginx/html -w /usr/share/nginx/html --name nginx-for-workshop nginx
```

* Check home page
```
curl localhost
```

### Test it again to check if the home page changed
```
curl localhost
```
### Access filesystem of docker container
```
# Access container
docker exec -it nginx-for-workshop /bin/bash
```
### Delete container
```
docker rm nginx-for-workshop
```
### Delete image
```
docker rmi -f nginx
```

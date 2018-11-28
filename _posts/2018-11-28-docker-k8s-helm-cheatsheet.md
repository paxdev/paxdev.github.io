---
  title: Docker/Kubernetes/Helm Cheat-Sheet
---

```bash
# build a dockerfile in current directory with name:tag {name:tag}
docker build -t {name:tag} .

# push to remote repo
docker tag {name:tag} {myrepo}/{name:tag}
docker push {myrepo}/{name:tag}

# list images
docker image ls #(--all)

# run image forwarding port 8080 to 80 and pass environment variable  
docker run -p 8080:80 -e SOMEPARAM="Some Value" {name:tag}

# list containers
docker ps #(--all)

# stop running container stop it
docker stop {first-few-chars-of-id}

# remove container 
docker rm {first-few-chars-of-id}
```

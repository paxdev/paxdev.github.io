---
  title: Docker/Kubernetes/Helm Cheat-Sheet
---

# Docker
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

# Helm
```bash
# Package a chart
helm package {chart-name} --version {version} # version must be semver 2
# outputs {chart-name}-{version}.tgz

# Install Helm chart {chart-name} (or {package}) with release name {release-name} to namespace {my-namespace} (uses upgrade --install to upgrade ... or install!)
# --recreate-pods will restart any running pods
# --set overrides a value in the values.yaml
# if uncommented, the -f flag will override values.yaml with a file
helm upgrade --install {release-name} {chart-name} --namespace {my-namespace} --recreate-pods --set replicaCount=2 # -f ./{some.values.release.yaml}

# get release status
helm status {release-name}

# remove helm release (note sometimes failed releases can get "stuck" and block newer releases)
helm delete {release-name} --purge
```

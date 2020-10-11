# Wolt - DevOps Assignment 1.2
- Author: Gerardo Prieto

## Requirements

- Kubernetes cluster
- Helm installed (https://helm.sh/docs/intro/install/)
- Nginx ingress controller enabled
- Docker installed

## Setup

- Install cert-manager:

```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.2/cert-manager.yaml 
```

- Install this docker-registry solution with Helm:

```
helm install docker-registry ./docker-registry
```

or by changing default config:

```
export REGISTRY_USERNAME=admin
export REGISTRY_PASSWORD=admin1234

helm install docker-registry ./docker-registry \
--set registry.dns=test.wolt.com \
--set registry.email=admin@wolt.com \
--set registry.username=$REGISTRY_USERNAME \
--set registry.password=$REGISTRY_PASSWORD \
--set registry.htpasswd=$(docker run --entrypoint htpasswd --rm httpd -Bbn $REGISTRY_USERNAME $REGISTRY_PASSWORD | base64) \
--set registry.storage=5Gi
```

_Note: `admin` / `admin1234` credentials should be changed for Production usage_


## Local docker-registry

```
kubectl port-forward service/docker-registry 5000 -n docker-registry
```

`
curl -u admin:admin1234 localhost.com:5000/v2/_catalog
`


## Runing in Production:

- A domain record pointing to the K8s cluster is needed (e.g: registry.wolt.com)
- `cert-manager` will automatically create a TLS certificate using Issuer and Certificate resources using Let's Encrypt

`
curl -u user:password https://registry.wolt.com/v2/_catalog
`


## Usage:

```
docker login https://registry.wolt.com -u admin -p admin1234
docker pull busybox:latest
docker tag busybox:latest registry.wolt.com/busybox:latest
docker push registry.wolt.com/busybox:latest
```

## Tear down

```
helm uninstall docker-registry
```

## Optional task:

### The Plan

- Different docker image repositories need to be pruned based on different conditions (latest X images, images starting with PR, keep only semantic releases, etc) and needs to be automated.

### The Solution

- Docker Registry Pruner image (https://github.com/tumblr/docker-registry-pruner) takes care of this by passing desired config file and docker-registry URL (See https://github.com/tumblr/docker-registry-pruner/blob/master/config/examples/example.yaml)
- A Kubernetes CronJob will do the work by calling this image every 12 hours and prune all images matching desired configuration

### Considerations

- docker-registry image deletion needs to be enabled. This is already done by default in this solution.
- If using AWS S3 for storage it's easier to do this by just using a clean up Lyfecicle Rule matching a particular object.


# Improvements / Ideas

- Added cert-manager for TLS support so it can be used in Production.
- For the garbage collect cron job I've set a crontab that performs `garbage-collect` internally in registry container rather than using a Kubernetes CronJob.
- AWS S3 is a better solution for storage as it's cheaper than Persistent Volumes.
- Using Terraform and/or Ansible could be a potential next step for this soultion, in terms of automation.


## License

Distributed under the MIT License. See `LICENSE` for more information.



## Contact

Gerardo Prieto - sherar@gmail.com

Project Link: [https://github.com/sherar/k8s-docker-registry](https://github.com/sherar/k8s-docker-registry)
<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

# K8s Docker Registry + Redis

K8s Docker Registry with Redis for image caching

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
--set registry.dns=test.yourcompany.com \
--set registry.email=admin@yourcompany.com \
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

- A domain record pointing to the K8s cluster is needed (e.g: registry.yourcompany.com)
- `cert-manager` will automatically create a TLS certificate using Issuer and Certificate resources using Let's Encrypt

`
curl -u user:password https://registry.yourcompany.com/v2/_catalog
`


## Usage:

```
docker login https://registry.yourcompany.com -u admin -p admin1234
docker pull busybox:latest
docker tag busybox:latest registry.yourcompany.com/busybox:latest
docker push registry.yourcompany.com/busybox:latest
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

Gerardo Prieto - [@sherarr](https://twitter.com/sherarr)

Project Link: [https://github.com/sherar/k8s-docker-registry-redis](https://github.com/sherar/k8s-docker-registry-redis)


<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/sherar/k8s-docker-registry-redis.svg?style=for-the-badge
[contributors-url]: https://github.com/sherar/k8s-docker-registry-redis/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/sherar/k8s-docker-registry-redis.svg?style=for-the-badge
[forks-url]: https://github.com/sherar/k8s-docker-registry-redis/network/members
[stars-shield]: https://img.shields.io/github/stars/sherar/k8s-docker-registry-redis.svg?style=for-the-badge
[stars-url]: https://github.com/sherar/k8s-docker-registry-redis/stargazers
[issues-shield]: https://img.shields.io/github/issues/sherar/k8s-docker-registry-redis.svg?style=for-the-badge
[issues-url]: https://github.com/sherar/k8s-docker-registry-redis/issues
[license-shield]: https://img.shields.io/github/license/sherar/k8s-docker-registry-redis.svg?style=for-the-badge
[license-url]: https://github.com/sherar/k8s-docker-registry-redis/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/gerardo-prieto/
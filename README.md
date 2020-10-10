# Wolt - DevOps Assignment 1.2
Author: Gerardo Prieto

## Requirements
- Kubernetes cluster
- Helm installed
- Nginx ingress controller enabled
- Docker installed

## Setup

- Install cert-manager:

```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.2/cert-manager.yaml 
```

- Install this docker-registry solution with Helm:

```
helm install docker-registry ./docker-registry \
		   --set emailContact=admin@wolt.com \
		   --set host=registry.wolt.com \
		   --set htpasswd=$(docker run --entrypoint htpasswd --rm httpd -Bbn admin admin1234 | base64) \
		   --set registryStorage=5Gi 
```

Note: `admin` / `admin1234` credentials should be changed for Production usage


## Local docker-registry

```
kubectl port-forward deployment/docker-registry 5000:5000 -n docker-registry
```


`
curl -u admin:admin1234 http://localhost.com/v2/_catalog
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


# Improvements / Ideas:
- AWS S3 is a better solution for storage as it's cheaper than Persistent Volumes

# Comments:
- For the garbage collect cron job I've set a crontab that performs `garbage-collect` internally in registry container rather than using a Kubernetes CronJob
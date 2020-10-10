Wolt - DevOps Assignment 1.2

## Requirements
- Kubernetes cluster
- Helm installed
- Nginx ingress controller enabled
- Docker installed

## Setup

- Install cert-manager

```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.2/cert-manager.yaml 
```

- Install docker-registry

```
helm install docker-registry ./docker-registry \
		   --set emailContact=admin@wolt.com \
		   --set host=registry.wolt.com \
		   --set htpasswd=$(docker run --entrypoint htpasswd --rm httpd -Bbn admin admin1234 | base64) \
		   --set registryStorage=5Gi 
```

Note: admin / admin1234 credentials should be changed for Production usage


## Local docker-registry

```
kubectl port-forward deployment/docker-registry 5000:5000 -n docker-registry
```

## Runing it in Production:

- A domain record pointing to the K8s cluster is needed (e.g: registry.wolt.com)
- `cert-manager` will automatically create a TLS certificate using Issuer and Certificate resources using Let's Encrypt


## Test:

Localhost:
`curl -u user:password http://localhost.com/v2/_catalog`

Production:
`curl -u user:password https://registry.wolt.com/v2/_catalog`


## Usage:

```
docker login https://registry.wolt.com -u admin -p admin1234
docker pull busybox:latest
docker tag busybox:latest registry.wolt.com/busybox:latest
docker push registry.wolt.com/busybox:latest
```


Improvements/Ideas:
- User AWS S3 for storage as it's cheaper than Persistent Volumes

Comments:
- For the garbage collect cron job I've set a crontab that performs `garbage-collect` internally in registry container
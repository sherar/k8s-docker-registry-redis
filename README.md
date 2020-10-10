Wolt Test

Consideraciones:
- User S3 for storage
- Domain pointint to K8s Cluster
- Instalar Cert Manager: kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.2/cert-manager.yaml 
- Para el cronjob se uso una crontab


Parametrizar:
- local.dev (DNS- Registry USER)
- Registry PASSWORD
- EMAIL for LetsEncrypt

Problemas:
- No SSL from localhost


TODO:
- Makefile con parametros / variables
- Escribir el readme mejor
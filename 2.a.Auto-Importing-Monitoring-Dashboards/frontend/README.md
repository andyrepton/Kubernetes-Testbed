# Frontend
## Cert-Manager

This uses the instructions at https://cert-manager.io/docs/installation/kubernetes/ , in case the yaml files in this directory are out of date

## Ingress controller

This uses the instructions at https://kubernetes.github.io/ingress-nginx/deploy/, in case the yaml files in this directory are out of date

## Important

If you are running on GKE, run the following before applying the files in this directory:

```kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole=cluster-admin \
    --user=$(gcloud config get-value core/account)```

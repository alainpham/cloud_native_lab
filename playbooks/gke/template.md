### TEMPLATESVC

```bash
export PROJECT_ARTIFACTID=TEMPLATESVC
export PROJECT_VERSION=1.0-SNAPSHOT
export CONTAINER_REGISTRY=alainpham
export KUBE_INGRESS_ROOT_DOMAIN=yourowndomain.duckdns.org

wget -O /tmp/TEMPLATESVC.yaml https://raw.githubusercontent.com/alainpham/hotel-platform-demo/master/TEMPLATESVC/src/main/kube/deploy.envsubst.yaml

envsubst < /tmp/TEMPLATESVC.yaml | kubectl delete -f -
envsubst < /tmp/TEMPLATESVC.yaml | kubectl apply -f -
```

Check out https://TEMPLATESVC.yourowndomain.duckdns.org/swagger-ui/index.html?url=/v3/api-docs
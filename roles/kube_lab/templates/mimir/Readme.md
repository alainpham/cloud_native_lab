helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm show values grafana/mimir-distributed

kubectl create ns mimir

helm template mimir grafana/mimir-distributed -n mimir -f kube-yaml/mimir/mimir-minio.yml > kube-yaml/mimir/mimir-minio.kube.yml

kubectl create ns gem

helm template gem grafana/mimir-distributed -n gem -f kube-yaml/mimir/gem-minio.yml > kube-yaml/mimir/gem-minio.kube.yml

helm install mimir grafana/mimir-distributed -n mimir -f kube-yaml/mimir/mimir-minio.yml

helm uninstall mimir  -n mimir
-----------------


helm template mimir grafana/mimir-distributed -n mimir -f large.yml > large.kube.

helm template mimir grafana/mimir-distributed -n mimir -f verycheap-ceph.yml > verycheap-ceph.kube.yml

helm install mimir grafana/mimir-distributed -n mimir -f verycheap-ceph.yml 
```

kubectl -n mimir get cm mimir-tsdb -o yaml | grep BUCKET_HOST | awk '{print $2}'
kubectl -n mimir get secret mimir-tsdb -o yaml | grep AWS_ACCESS_KEY_ID | awk '{print $2}' | base64 --decode
kubectl -n mimir get secret mimir-tsdb -o yaml | grep AWS_SECRET_ACCESS_KEY | awk '{print $2}' | base64 --decode

kubectl -n mimir get cm mimir-ruler -o yaml | grep BUCKET_HOST | awk '{print $2}'
kubectl -n mimir get secret mimir-ruler -o yaml | grep AWS_ACCESS_KEY_ID | awk '{print $2}' | base64 --decode
kubectl -n mimir get secret mimir-ruler -o yaml | grep AWS_SECRET_ACCESS_KEY | awk '{print $2}' | base64 --decode

kubectl -n mimir get cm mimir-ent-admin -o yaml | grep BUCKET_HOST | awk '{print $2}' 
kubectl -n mimir get secret mimir-ent-admin -o yaml | grep AWS_ACCESS_KEY_ID | awk '{print $2}' | base64 --decode
kubectl -n mimir get secret mimir-ent-admin -o yaml | grep AWS_SECRET_ACCESS_KEY | awk '{print $2}' | base64 --decode

```
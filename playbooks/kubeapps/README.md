
- [General vars](#general-vars)
- [Minio](#minio)
- [Kafka](#kafka)

# General vars

```bash 
export KUBE_INGRESS_ROOT_DOMAIN=cxw.duckdns.org

```

# Minio

```bash
export IMAGES_BUSYBOX=$(/snap/bin/yq -r '.images.busybox' ./../vars/images.yml )
export IMAGES_MINIO=$(/snap/bin/yq -r '.images.minio' ./../vars/images.yml )
export BUCKETS="mimir-tsdb mimir-ruler mimir-alertmanager mimir-admin loki-chunks loki-admin loki-ruler"

envsubst < templates/minio.envsubst.yml | kubectl delete -f -
envsubst < templates/minio.envsubst.yml | kubectl apply -f -

```

# Kafka
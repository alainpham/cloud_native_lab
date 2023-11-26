

```
kubectl -n ingress-nginx create  secret tls nginx-ingress-tls  --key="/home/${USER}/apps/tls/cfg/live/alainpham.duckdns.org/privkey.pem"   --cert="/home/${USER}/apps/tls/cfg/live/alainpham.duckdns.org/fullchain.pem"  --dry-run=client -o yaml | kubectl apply -f -


kubectl apply -f playbooks/gke/ingress.yaml


export MIMIR_URL=YOURURL
export MIMIR_USR=YOURUSER
export MIMIR_KEY=YOURKEY

export LOKI_URL=YOURURL
export LOKI_USR=YOURUSR
export LOKI_KEY=YOURKEY

export TEMPO_URL=YOURURL
export TEMPO_USR=YOURUSER
export TEMPO_KEY=YOURKEY

envsubst < otelcol.yaml | kubectl -n gcloud apply -f -

kubectl apply -f app.yaml
```
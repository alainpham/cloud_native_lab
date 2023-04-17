export NAMESPACE=monitoring
kubectl create ns $NAMESPACE
envsubst < kube-yaml/monitoring/prometheus-local.yaml | kubectl apply -f -
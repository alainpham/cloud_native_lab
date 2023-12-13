# Open Telemetry & Grafana Application Observability (Loki, Prometheus / Mimir, Tempo)

- [Open Telemetry \& Grafana Application Observability (Loki, Prometheus / Mimir, Tempo)](#open-telemetry--grafana-application-observability-loki-prometheus--mimir-tempo)
  - [Application Architecture](#application-architecture)
  - [Kubernetes Prerequisites (optional)](#kubernetes-prerequisites-optional)
    - [Generate a domain name and wildcard certificates with Letsencrypt](#generate-a-domain-name-and-wildcard-certificates-with-letsencrypt)
    - [Create ingress router](#create-ingress-router)
  - [Deploy OTEL Collector sending data to Grafana Cloud](#deploy-otel-collector-sending-data-to-grafana-cloud)
  - [Deploy apps](#deploy-apps)
    - [broker](#broker)
    - [availability-service](#availability-service)
    - [booking-hub](#booking-hub)
    - [notification-service](#notification-service)
    - [email](#email)
    - [sms](#sms)
    - [k6 load testing](#k6-load-testing)


This is a guide to deploy a set of synchronous and asynchronous microservices to observed with & Grafana Cloud instance.

The Services are written in Java Spring boot and their source code can be found [here](https://github.com/alainpham/hotel-platform-demo).

## Application Architecture

```
      +--------------+                                                                 
      | client-calls |                                                                 
      +--------------+                                                                 
            |  ^                                                                       
            |  |                                                                       
            v  |                                                                       
      +-------------+         +--------------------------+       +-------------+       
      | booking-hub | <-----> |   notification-service   | ----> |   broker    |       
      +-------------+         +--------------------------+       +-------------+       
            |  ^                                                   |          |        
            |  |                                                   |          |        
            |  |                                                   v          v        
            v  |                                             +-------+      +---------+
+------------------------+                                   | email |      |   sms   |
|  availability-service  |                                   +-------+      +---------+
+------------------------+                                                             
```

## Kubernetes Prerequisites (optional)

The kube deployment yaml files use ingress, to expose application http(s) services to ouside of kubernetes through a loadbalancer type service.

If you want to have clean certificates with valid letsencrypt and a proper domain name follow these optional steps. I use the free duckdns.org dns service.

For this demo to work you this section is optional

this is the architecture we will be building

```
     +----------------------------------------------+         
     |                                              |         
     |   https://*.yourowndomain.duckdns.org        |         
     |                                              |         
     |   with Letsencrypt TLS wildcard certificate  |         
     |                                              |         
     +----------------------------------------------+         
                             |                                
                             |                                
+-----------------------------------------------------------+ 
| Kubernetes Service of Type Loadbalancer of Nginx Ingress  | 
+-----------------------------------------------------------+ 
                             |                                
                             |                                
          +-------------------------------------+             
          |  Kubernetes Ingress of Application  |             
          +-------------------------------------+             
                             |                                
                             |                                
           +---------------------------------------+          
           | Kubernetes Service of type ClusterIP  |          
           +---------------------------------------+          
                             |                                
                             |                                
                   +---------------------+                    
                   |   Kubernetes Pods   |                    
                   +---------------------+                    
```

### Generate a domain name and wildcard certificates with Letsencrypt

Go to duckdns.org and create a domain name as in yourowndomain.duckdns.org. You don't need to specify an IP address yet. We will point it to the 
All applications deployed on kube will be accessible with an url like appname.yourowndomain.duckdns.org

```bash
export DT=xxxx-xxxx-xxx-xxx-xxxxx
export EMAIL=xxx@yyy.com
export WILDCARD_DOMAIN=*.yourowndomain.duckdns.org

docker run -v "$(pwd)/sensitive/letsencrypt/data:/etc/letsencrypt" -v "$(pwd)/sensitive/letsencrypt/logs:/var/log/letsencrypt" infinityofspace/certbot_dns_duckdns:v1.3 \
   certonly \
     --non-interactive \
     --agree-tos \
     --email ${EMAIL} \
     --preferred-challenges dns \
     --authenticator dns-duckdns \
     --dns-duckdns-token ${DT} \
     --dns-duckdns-propagation-seconds 15 \
     -d "${WILDCARD_DOMAIN}"
```

### Create ingress router

```bash
kubectl create ns ingress-nginx

sudo kubectl -n ingress-nginx create  secret tls nginx-ingress-tls  --key="$(pwd)/sensitive/letsencrypt/data/live/yourowndomain.duckdns.org/privkey.pem"   --cert="$(pwd)/sensitive/letsencrypt/data/live/yourowndomain.duckdns.org/fullchain.pem"  --dry-run=client -o yaml | kubectl apply -f -

# on GKE you can create ingress using LB
kubectl apply -f ingress-lb.yaml

# with local clusters like kind or other supporting hostport you can use this
kubectl apply -f ingress-hostport.yaml
```

## Deploy OTEL Collector sending data to Grafana Cloud

```bash
export MIMIR_URL=https://prometheus-prod-01-eu-west-0.grafana.net/api/prom/push
export MIMIR_USR=xxxxx
export MIMIR_KEY=xxxxx

export LOKI_URL=https://logs-prod-eu-west-0.grafana.net/loki/api/v1/push
export LOKI_USR=xxxxx
export LOKI_KEY=xxxxx

export TEMPO_URL=tempo-eu-west-0.grafana.net:443
export TEMPO_USR=xxxxx
export TEMPO_KEY=xxxxx

kubectl create ns agents

wget -O /tmp/otelcol.yaml https://raw.githubusercontent.com/alainpham/cloud_native_lab/master/playbooks/gke/otelcol.yaml
envsubst < /tmp/otelcol.yaml | kubectl -n agents apply -f -

```

## Deploy apps

### broker

```bash
export CONTAINER_REGISTRY=apache
export PROJECT_ARTIFACTID=activemq-artemis
export PROJECT_VERSION=2.31.2
export KUBE_INGRESS_ROOT_DOMAIN=yourowndomain.duckdns.org

kubectl create ns broker

wget -O /tmp/broker.yaml https://raw.githubusercontent.com/alainpham/hotel-platform-demo/master/broker/broker.envsubst.yaml

envsubst < /tmp/broker.yaml | kubectl delete -n broker -f -
envsubst < /tmp/broker.yaml | kubectl apply -n broker -f -
```

### availability-service

```bash
export PROJECT_ARTIFACTID=availability-service
export PROJECT_VERSION=1.0-SNAPSHOT
export CONTAINER_REGISTRY=alainpham
export KUBE_INGRESS_ROOT_DOMAIN=yourowndomain.duckdns.org

wget -O /tmp/availability-service.yaml https://raw.githubusercontent.com/alainpham/hotel-platform-demo/master/availability-service/src/main/kube/deploy.envsubst.yaml

envsubst < /tmp/availability-service.yaml | kubectl delete -f -
envsubst < /tmp/availability-service.yaml | kubectl apply -f -
```

Check out https://availability-service.yourowndomain.duckdns.org/swagger-ui/index.html?url=/v3/api-docs

### booking-hub

```bash
export PROJECT_ARTIFACTID=booking-hub
export PROJECT_VERSION=1.0-SNAPSHOT
export CONTAINER_REGISTRY=alainpham
export KUBE_INGRESS_ROOT_DOMAIN=yourowndomain.duckdns.org

wget -O /tmp/booking-hub.yaml https://raw.githubusercontent.com/alainpham/hotel-platform-demo/master/booking-hub/src/main/kube/deploy.envsubst.yaml

envsubst < /tmp/booking-hub.yaml | kubectl delete -f -
envsubst < /tmp/booking-hub.yaml | kubectl apply -f -
```

Check out https://booking-hub.yourowndomain.duckdns.org/swagger-ui/index.html?url=/v3/api-docs

### notification-service

```bash
export PROJECT_ARTIFACTID=notification-service
export PROJECT_VERSION=1.0-SNAPSHOT
export CONTAINER_REGISTRY=alainpham
export KUBE_INGRESS_ROOT_DOMAIN=yourowndomain.duckdns.org

wget -O /tmp/notification-service.yaml https://raw.githubusercontent.com/alainpham/hotel-platform-demo/master/notification-service/src/main/kube/deploy.envsubst.yaml

envsubst < /tmp/notification-service.yaml | kubectl delete -f -
envsubst < /tmp/notification-service.yaml | kubectl apply -f -
```

Check out https://notification-service.yourowndomain.duckdns.org/swagger-ui/index.html?url=/v3/api-docs



### email

```bash
export PROJECT_ARTIFACTID=email
export PROJECT_VERSION=1.0-SNAPSHOT
export CONTAINER_REGISTRY=alainpham
export KUBE_INGRESS_ROOT_DOMAIN=yourowndomain.duckdns.org

wget -O /tmp/email.yaml https://raw.githubusercontent.com/alainpham/hotel-platform-demo/master/email/src/main/kube/deploy.envsubst.yaml

envsubst < /tmp/email.yaml | kubectl delete -f -
envsubst < /tmp/email.yaml | kubectl apply -f -
```

Check out https://email.yourowndomain.duckdns.org/swagger-ui/index.html?url=/v3/api-docs

### sms

```bash
export PROJECT_ARTIFACTID=sms
export PROJECT_VERSION=1.0-SNAPSHOT
export CONTAINER_REGISTRY=alainpham
export KUBE_INGRESS_ROOT_DOMAIN=yourowndomain.duckdns.org

wget -O /tmp/sms.yaml https://raw.githubusercontent.com/alainpham/hotel-platform-demo/master/sms/src/main/kube/deploy.envsubst.yaml

envsubst < /tmp/sms.yaml | kubectl delete -f -
envsubst < /tmp/sms.yaml | kubectl apply -f -
```

Check out https://sms.yourowndomain.duckdns.org/swagger-ui/index.html?url=/v3/api-docs


### k6 load testing

```bash
kubectl apply -f https://raw.githubusercontent.com/alainpham/hotel-platform-demo/master/k6/k6.yaml
```
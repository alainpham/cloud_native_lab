# Open Telemetry & Grafana Application Observability (Loki, Prometheus / Mimir, Tempo)

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

```
export DUCKTOKEN=xxxx-xxxx-xxx-xxx-xxxxx
export EMAIL=xxx@yyy.com
export WILDCARD_DOMAIN=*.yourowndomain.duckdns.org

docker run -v "$(pwd)/sensitive/letsencrypt/data:/etc/letsencrypt" -v "$(pwd)/sensitive/letsencrypt/logs:/var/log/letsencrypt" infinityofspace/certbot_dns_duckdns:v1.3 \
   certonly \
     --non-interactive \
     --agree-tos \
     --email ${EMAIL} \
     --preferred-challenges dns \
     --authenticator dns-duckdns \
     --dns-duckdns-token ${DUCKTOKEN} \
     --dns-duckdns-propagation-seconds 15 \
     -d "${WILDCARD_DOMAIN}"
```

```
kubectl -n ingress-nginx create  secret tls nginx-ingress-tls  --key="/home/${USER}/apps/tls/cfg/live/vrbx.duckdns.org/privkey.pem"   --cert="/home/${USER}/apps/tls/cfg/live/vrbx.duckdns.org/fullchain.pem"  --dry-run=client -o yaml | kubectl apply -f -

kubectl apply -f ingress-lb.yaml
or
kubectl apply -f ingress-hostport.yaml


export MIMIR_URL=YOURURL
export MIMIR_USR=YOURUSER
export MIMIR_KEY=YOURKEY

export LOKI_URL=YOURURL
export LOKI_USR=YOURUSR
export LOKI_KEY=YOURKEY

export TEMPO_URL=YOURURL
export TEMPO_USR=YOURUSER
export TEMPO_KEY=YOURKEY

envsubst < otelcol.yaml | kubectl -n agents apply -f -

kubectl apply -f app.yaml
```
# For Loki Create 9 vms

2 read vms for querier and query frontend
3 backend vms for the ruler
3 write vms for distributor and ingester
1 for infra

```
debianimage=debian-11-genericcloud-amd64-20230124-1270

vmcreate gelmmaster 1048 2 $debianimage 50 40G 1G debian11

vmcreate backend01 1048 2 $debianimage 56 40G 1G debian11
vmcreate backend02 1048 2 $debianimage 57 40G 1G debian11
vmcreate backend03 1048 2 $debianimage 58 40G 1G debian11

dvm infra

dvm read01
dvm read02

dvm write01
dvm write02
dvm write03

dvm backend01
dvm backend02
dvm backend03
```




```

sudo su - enterprise-logs

cd /var/lib/enterprise-logs

bash

/usr/local/bin/enterprise-logs \
              -config.file=/etc/enterprise-logs/config-mono.yaml \
              -target=all


```



```

TOKEN=ZGV2LWFkbWluLWRldi1hZG1pbi10a24yOkpgLHguWzAzaWA2OSMxMWshezU1MDUlLA==


current_date=`date +%s%N` && \
curl -u dev:$TOKEN infra:3100/loki/api/v1/push \
-H "Content-Type: application/json" \
-H "X-Scope-OrdID: dev" \
--data "{\"streams\": [{ \"stream\": { \"job\": \"example\" }, \"values\": [ [ \"$current_date\", \"A log line\" ] ] }]}"
```
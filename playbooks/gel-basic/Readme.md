# For Loki Create 9 vms

3 read nodes
3 write write nodes
1 for infra

```console
debianimage=debian-11-genericcloud-amd64-20230124-1270

vmcreate infra 1048 2 $debianimage 50 40G 1G debian11

vmcreate read01 1048 2 $debianimage 60 40G 1G debian11
vmcreate read02 1048 2 $debianimage 61 40G 1G debian11
vmcreate read03 1048 2 $debianimage 62 40G 1G debian11

vmcreate write01 1048 2 $debianimage 70 40G 1G debian11
vmcreate write02 1048 2 $debianimage 71 40G 1G debian11
vmcreate write03 1048 2 $debianimage 72 40G 1G debian11


dvm infra

dvm read01
dvm read02
dvm read03

dvm write01
dvm write02
dvm write03

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

current_date=`date +%s%N`
TOKEN=ZGV2LWFsbC1hY2Nlc3MtZGV2LWFsbC1hY2Nlc3MtdGtuOldDbH05NjI6MzQ2ezY6LyMqPTUuLXg4OA==


curl -u dev:$TOKEN infra:8080/loki/api/v1/push \
-H "Content-Type: application/json" \
-H "X-Scope-OrdID: dev" \
--data "{\"streams\": [{ \"stream\": { \"job\": \"example\" }, \"values\": [ [ \"$current_date\", \"A log line\" ] ] }]}"
```
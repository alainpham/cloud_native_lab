# For Loki Create 9 vms

2 read vms for querier and query frontend
3 backend vms for the ruler
3 write vms for distributor and ingester
1 for infra

```console
debianimage=debian-11-genericcloud-amd64-20230124-1270

vmcreate infra 1048 2 $debianimage 50 40G 1G debian11

vmcreate read01 1048 2 $debianimage 51 40G 1G debian11
vmcreate read02 1048 2 $debianimage 52 40G 1G debian11

vmcreate write01 1048 2 $debianimage 53 40G 1G debian11
vmcreate write02 1048 2 $debianimage 54 40G 1G debian11
vmcreate write03 1048 2 $debianimage 55 40G 1G debian11

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
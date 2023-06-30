# Install Vanilla Kubernetes 1.25.11


OS : Debian 12
Kubernetes version : 1.25.11

## Prepare hosts

```console

export kube_version=1.25.11

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

sudo apt install apt-transport-https gnupg2 curl rsync lsb-release


cat <<EOF | sudo tee /etc/ssl/certs/root-ca.pem
-----BEGIN CERTIFICATE-----
MIIFDTCCAvWgAwIBAgIUK0zhekgHfcVNg/D7P7pEgyeVMMgwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAwwKKi53b3JrLmxhbjAgFw0yMjA2MjYyMjE1MjdaGA8zMDIx
MTAyNzIyMTUyN1owFTETMBEGA1UEAwwKKi53b3JrLmxhbjCCAiIwDQYJKoZIhvcN
AQEBBQADggIPADCCAgoCggIBAMRU7VKP2NSR9C/FvBh67bVoHyqkvZuaQfOaS8aB
cnuB5a25yA4a0VPvqGolfzDYOVtXIQ1Tq+LiTuB4HnNI0ghhpFKXL04MDbnN+Cxb
WOpEsQ2grFN7Tkt4NmqJe3Y2iwGkCqRBZhGyY4h47KYv2yW6rUznI1Ix9MTquMc9
P5OQS+eCp77bL3IKdXNK6XyHHt4oukPMDmfSX4VCnQ/EEkUaZf/KBoChUxVPJJiS
nXL9YOLAtWwg9oe7ISS2n6bX+tVFnkXE+yFcCkzabE+ceXi+C0ZMSYyspadEFZ9X
NvcaCvogweLKGa6SU1ZczPU9hJXso4nJN0ibR9t7GV/WZITzuT+CfA51/8HZu9kV
iT/VyK4QORgvkobnYOY3ZQt8veqvGcxMTzmwZycqViSgafZdm9/jANrSyle76oax
ok2G+62e0TDFI1mAKjCLLR0onihfBYoVBqnChuTwHH3CWo7/WtAzncWLLRXbaF6+
yPpIEOdSk0N2OzF0y0JSUHZGOSPKwTkMjCb6wm/OYP9Nst8ejd0XCrMwweoQ9T7E
Vu420R0/EtEdHDnNihb5g9ZhH28wUUVOM+wx4+ms2JyrpK90p3Ln8xQBkHkNOW2z
KW5m5uW+L4fC9FDW0Wm/zQ3TznG9dz9TFXuYg1t2o79vrBIbYAjaDgyA1M9CM1SG
QlCtAgMBAAGjUzBRMB0GA1UdDgQWBBTBah+HM4fUHo0monk/aPfjcuPBLDAfBgNV
HSMEGDAWgBTBah+HM4fUHo0monk/aPfjcuPBLDAPBgNVHRMBAf8EBTADAQH/MA0G
CSqGSIb3DQEBCwUAA4ICAQBgaDFjQxXn7kwLF6zz1aEEAaukWi4mCcMcT/9Wmgpf
lKjINOteleIssnDuIm1rKfuiJq5Od2Xoz5KFNtvSBz4XtPnJ98T5SXhG2TdarvLu
1FYFIO/JuPga2acsWW/uDG5Ap/kxeXBxoMclERHUj936dxLgK658WeI/euOQAnqQ
i9zsUY4Olew6F9biK05/KJZXCcWTJrNdmSZwobGmB+8vjcHYcwwf2Am2OsI0K61R
CbloRg00gzCxdvapzfHjHzqtrknOlTIWrSWV7i572p7xRoHlQpu+avNFGlRfYRr8
6OnnwwgHaI4SnNKyOsJE2RXgtorkhHxCmaRnGIN/cec+sQnVad5GpnMHdouXUyLD
jix41UxjZX8GTgIdRcssCvljhfM3e/D3moDTc1MKF/gXhLHJZmsl7BMPa19IU4Rh
W12prSAlfmU4JmMWB3T1/qZitsseegE01bQnwz6YENovHJeEIqm+Ytn/w8VWvFvI
aY8kMBEjMRSDC7V+mUkF0h0CLGyv6lCGrwNRKAq1L0+gYaJUYCPXXvfmHvLRYSzm
Giz8GJpAr5SHpJzKQF2ujnMiC8G0GJEkVycqcqRUVvoXbv/Ao+SZ9IP+rpi2juK2
lGUZlcn5XP2zoHXbygziRV+A7JFt6jKxecTKgdEBuMEmacZvETK5EXqeUiM5C3OJ
Jw==
-----END CERTIFICATE-----

EOF


sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list


curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install containerd.io kubeadm=${kube_version}-00 kubelet=${kube_version}-00 kubectl=${kube_version}-00


cat <<EOF | sudo tee /etc/containerd/config.toml
disabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd"
temp = ""
version = 2

[cgroup]
  path = ""

[debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0

[metrics]
  address = ""
  grpc_histogram = false

[plugins]

  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"

  [plugins."io.containerd.grpc.v1.cri"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_image_defined_volumes = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "registry.k8s.io/pause:3.6"
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""

    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1

    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"

      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true

      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"

    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""

  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"

  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"

  [plugins."io.containerd.internal.v1.tracing"]
    sampling_ratio = 1.0
    service_name = "containerd"

  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"

  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false

  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false

  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false

  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]

  [plugins."io.containerd.service.v1.tasks-service"]
    rdt_config_file = ""

  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    root_path = ""
    upperdir_label = false

  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""

  [plugins."io.containerd.tracing.processor.v1.otlp"]
    endpoint = ""
    insecure = false
    protocol = ""

[proxy_plugins]

[stream_processors]

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"

[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[ttrpc]
  address = ""
  gid = 0
  uid = 0

EOF

sudo systemctl restart containerd

```

## Create master node

```
export localreg=registry.awon.lan

kubeadm init --image-repository $localreg --pod-network-cidr=10.244.0.0/16

curl https://awoo.lan:6443/version -k

kubectl apply -f kube-yaml/flannel/flannel.yml
```

## ingress

```

kubectl create ns ingress-nginx  --dry-run=client -o yaml | kubectl apply -f -


kubectl -n ingress-nginx create  secret tls nginx-ingress-tls  --key="/home/apham/apps/tls/awoo.lan.key"   --cert="/home/apham/apps/tls/awoo.lan.crt"  --dry-run=client -o yaml | kubectl apply -f -

kubectl apply -f kube-yaml/nginx-ingress/nginx.yml


```

## metrics server

```console

kubectl apply -f kube-yaml/metrics-server/components.yaml

```

## localstorage

```console

kubectl apply -f kube-yaml/local-path-provisioner/local-path-storage.yaml

export DOMAIN=awoo.lan
envsubst < kube-yaml/minio/minio-var.yml | kubectl apply -f -
```



# Archived
OS : Debian 10
Current version : 1.25.3


```
sudo apt update
sudo apt upgrade

sudo apt -y install apt-transport-https gnupg2 curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
curl -s https://download.docker.com/linux/debian/gpg | sudo apt-key add -


echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
echo "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker-ce.list

sudo apt update

sudo apt -y install containerd.io
sudo apt -y install kubeadm kubelet kubectl

```




prepull images in a local repo

```
curl -s https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages | grep Version | awk '{print $2}'

sudo kubeadm config images list
```



```
images=(
   "curlimages/curl:7.85.0"
)

images=(
   # Kube
   registry.k8s.io/kube-apiserver:v1.26.0
   registry.k8s.io/kube-controller-manager:v1.26.0
   registry.k8s.io/kube-scheduler:v1.26.0
   registry.k8s.io/kube-proxy:v1.26.0
   registry.k8s.io/pause:3.9
   registry.k8s.io/etcd:3.5.6-0
   registry.k8s.io/coredns/coredns:v1.9.3


   # Flannel
   "flannelcni/flannel-cni-plugin:v1.1.0"
   "flannelcni/flannel:v0.20.0"

   # ingress nginx
   "registry.k8s.io/ingress-nginx/controller:v1.4.0"
   "registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343"

   # kube ui
   "kubernetesui/metrics-scraper:v1.0.8"
   "kubernetesui/dashboard:v2.6.1"

   # Open EBS
   "openebs/node-disk-manager:2.0.0"
   "openebs/node-disk-exporter:2.0.0"
   "openebs/provisioner-localpv:3.3.0"
   "openebs/node-disk-operator:2.0.0"
   "openebs/linux-utils:3.3.0"

   # Minio
   "minio/minio:RELEASE.2022-10-20T00-55-09Z"
   "docker.io/busybox:1.36.0"
   "curlimages/curl:8.00.1"

   # Grafana stack
   "grafana/enterprise-metrics:v2.3.0"
   "grafana/mimir:2.3.1"
   "docker.io/memcached:1.6.16-alpine"
   "prom/memcached-exporter:v0.6.0"

   "grafana/enterprise-logs:v1.5.2"
   "grafana/loki:2.6.1"

   "grafana/enterprise-traces:v1.3.0"
   "grafana/tempo:1.4.1"

   "grafana/grafana:9.2.1"
   "grafana/grafana-enterprise:9.2.1"
)

localrepo=registry.work.lan

for item in "${images[@]}"
do
   echo  $item
done


for item in "${images[@]}"
do
   localname=`echo "$item"| grep -o '[^/]*$' `
   echo ${localrepo}/${localname}
   skopeo copy docker://"$item" docker://${localrepo}/${localname}
done


for item in "${images[@]}"
do
   docker pull $item
done

for item in "${images[@]}"
do
   localname=`echo "$item"| grep -o '[^/]*$' `
   echo ${localrepo}/${localname}
   docker tag $item ${localrepo}/${localname}
done

for item in "${images[@]}"
do
   localname=`echo "$item"| grep -o '[^/]*$' `
   echo ${localrepo}/${localname}
   docker push ${localrepo}/${localname}
done


```


COnfigure container d

```

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system


sudo mkdir -p /etc/containerd


containerd config default | sudo tee /etc/containerd/config.toml

modify config.toml like this


    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."registry.work.lan".tls]
          insecure_skip_verify = true

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugin."io.containerd.grpc.v1.cri".registry.mirrors."registry.work.lan"]
          endpoint = ["https://registry.work.lan"]


copy pem file of registry to /etc/ssl/certs/ on master


sudo systemctl restart containerd

sudo reboot now

sudo kubeadm config images pull --image-repository registry.work.lan 

sudo kubeadm init --image-repository registry.work.lan

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

kubectl label node master node-role.kubernetes.io/role=master

kubectl get po --all-namespaces
kubectl get nodes

```

# Sandboxing

```

vmcreate sandbox 4096 4 debian-10-genericcloud-amd64 50 40G debian10

```
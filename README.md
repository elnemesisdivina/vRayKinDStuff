# vRayKinDStuff
This repo will try to explain the step zero towards the practice of K8s on your own mac :) folks with Linux distro can follow as just some commands need to be adapted. all of this based on this list:

##### K8s clusters on Docker Containers
-[KinD](https://kind.sigs.k8s.io/)
##### CNI
-[Antrea](https://antrea.io/)
##### L4 LB
-[MetalLB](https://metallb.universe.tf/)
##### K8s Management tool kit
-[Octant](https://reference.octant.dev/?path=/docs/docs-intro--page#getting-started)
#### k8s apps manager
-[Helm](https://helm.sh/)
-[Kubeapps](https://kubeapps.com/)
##### curated Apps catalog (Bitnami)
-[Tanzu Application Catalog](https://bitnami.com/)
##### Observability with steroids!
-[Tanzu Observability](https://docs.wavefront.com/)
##### Policy enforcement adn K8s cluster managment:
-[Tanzu Mission Control](https://docs.vmware.com/en/VMware-Tanzu-Mission-Control/index.html)
##### Continous Delivery 
-[ArgoCD](https://argoproj.github.io/argo-cd/)
###### Work in progress...
##### Continous Delivery
-[Concourse](https://concourse-ci.org/)
##### Container Registry
-[Harbor](https://goharbor.io/docs/2.2.0/install-config/)
-[Tanzu Service Mesh]
##### Other cluster with cillium CNI
##### Experiment with Multus

## Setting Up Kind on your Mac

**Step 0:** is to install `docker` on your mac

`https://docs.docker.com/docker-for-mac/install/`

##### Check if this fine (Client & Server)

```
docker version
```

**Step 1:** the is not a special sauce just install `kubectl`

```shell
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

``` 

 - Docker
 - Kind
 - Kubectl
 -
**Step 3:** Check to make sure kubectl is running

`kubectl version --client --short`

ourput example:

```shell
Client Version: v1.20.0
```
**Step 4:** Install [KinD](https://kind.sigs.k8s.io/)

```shell
brew install kind
```
or 

```shell
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.10.0/kind-darwin-amd64

```

**Step 4:** Create your config file using `vim` this will be use for `KinD` on how to crete your K8s cluster

`vim kind-antrea.yaml`

Edit and add your setting, in my case I decide to add `Antrea` CNI from the beginning, `kind-config.yaml`

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4 #always check api version
networking:
  disableDefaultCNI: true #disable kindet 
  podSubnet: 192.168.0.0/16
nodes: #set 1 master and 3 worker nodes
- role: control-plane
- role: worker
- role: worker
- role: worker
```
**Step 4:** Create your `KinD` cluster

`kind create cluster --name ernesto --config antrea-config.yaml`

Output fo this command example:

```shell
Creating cluster "ernesto" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼 
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-ernesto"
You can now use your cluster with:

kubectl cluster-info --context kind-ernesto

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```

run to check your recent create Docker cntrs aka Kubernetes Nodes:

`docker ps`

```shell
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                       NAMES
7964d8be4147   kindest/node:v1.19.1   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes   127.0.0.1:55961->6443/tcp   ernesto-control-plane
6950cf5d06b8   kindest/node:v1.19.1   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes                               ernesto-worker2
049e84df4540   kindest/node:v1.19.1   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes                               ernesto-worker3
0c742cd2e5cf   kindest/node:v1.19.1   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes                               ernesto-worker
```

`kind get clusters`

output example:

```shell
ernesto
```

`kubectl config view`

```shell
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:55961
  name: kind-ernesto
contexts:
- context:
    cluster: kind-ernesto
    user: kind-ernesto
  name: kind-ernesto
current-context: kind-ernesto
kind: Config
preferences: {}
users:
- name: kind-ernesto
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```
#Export kubeconfig of new created cluster on kind

`export KUBECONFIG="$(kind get kubeconfig --name="ernesto")"`


`kubectl get nodes`

```shell
NAME                    STATUS     ROLES    AGE     VERSION
ernesto-control-plane   NotReady   master   3m12s   v1.19.1
ernesto-worker          NotReady   <none>   2m38s   v1.19.1
ernesto-worker2         NotReady   <none>   2m37s   v1.19.1
ernesto-worker3         NotReady   <none>   2m37s   v1.19.1
```

`kubectl get pods -A`

```shell
NAMESPACE            NAME                                            READY   STATUS    RESTARTS   AGE
kube-system          coredns-f9fd979d6-7bbs5                         0/1     Pending   0          3m45s
kube-system          coredns-f9fd979d6-r9lz4                         0/1     Pending   0          3m45s
kube-system          etcd-ernesto-control-plane                      1/1     Running   0          3m58s
kube-system          kube-apiserver-ernesto-control-plane            1/1     Running   0          3m58s
kube-system          kube-controller-manager-ernesto-control-plane   1/1     Running   0          3m58s
kube-system          kube-proxy-5ttl4                                1/1     Running   0          3m32s
kube-system          kube-proxy-pqsgg                                1/1     Running   0          3m33s
kube-system          kube-proxy-r2t6j                                1/1     Running   0          3m45s
kube-system          kube-proxy-vk9qd                                1/1     Running   0          3m32s
kube-system          kube-scheduler-ernesto-control-plane            1/1     Running   0          3m58s
local-path-storage   local-path-provisioner-78776bfc44-9kw74         0/1     Pending   0          3m45s
```

***** Just in case you need it this is the way to delete the cluster 

`kind delete cluster --name ernesto`

**Step 4:** Config `Antrea` control plane and Dataplane (Antrea CNI for K8s)

Check also `https://antrea.io/docs/v0.11.1/kind/` for specifics on how to deploy antrea CNI on `KinD` 
TL;DR: "the OVS netdev datapath dopes not support TX checksumm offloading here is the link ref. if you like to goo deeper `https://github.com/vmware-tanzu/antrea/issues/14`" 

cloning the `Antrea` Repo

`git clone https://github.com/vmware-tanzu/antrea.git`

excecute teh fix to the nodes:

`kind get nodes --name ernesto | xargs ~/antrea/hack/kind-fix-networking.sh`

output sample:

```shell
Disabling TX checksum offload for node ernesto-control-plane (veth9e3b089)
Actual changes:
tx-checksumming: off
	tx-checksum-ip-generic: off
	tx-checksum-sctp: off
tcp-segmentation-offload: off
	tx-tcp-segmentation: off [requested on]
	tx-tcp-ecn-segmentation: off [requested on]
	tx-tcp-mangleid-segmentation: off [requested on]
	tx-tcp6-segmentation: off [requested on]
net.ipv4.tcp_retries2 = 4
Disabling TX checksum offload for node ernesto-worker2 (veth0e96cf2)
Actual changes:
tx-checksumming: off
	tx-checksum-ip-generic: off
	tx-checksum-sctp: off
tcp-segmentation-offload: off
	tx-tcp-segmentation: off [requested on]
	tx-tcp-ecn-segmentation: off [requested on]
	tx-tcp-mangleid-segmentation: off [requested on]
	tx-tcp6-segmentation: off [requested on]
net.ipv4.tcp_retries2 = 4
Disabling TX checksum offload for node ernesto-worker3 (veth4191ff6)
Actual changes:
tx-checksumming: off
	tx-checksum-ip-generic: off
	tx-checksum-sctp: off
tcp-segmentation-offload: off
	tx-tcp-segmentation: off [requested on]
	tx-tcp-ecn-segmentation: off [requested on]
	tx-tcp-mangleid-segmentation: off [requested on]
	tx-tcp6-segmentation: off [requested on]
net.ipv4.tcp_retries2 = 4
Disabling TX checksum offload for node ernesto-worker (veth2aeab04)
Actual changes:
tx-checksumming: off
	tx-checksum-ip-generic: off
	tx-checksum-sctp: off
tcp-segmentation-offload: off
	tx-tcp-segmentation: off [requested on]
	tx-tcp-ecn-segmentation: off [requested on]
	tx-tcp-mangleid-segmentation: off [requested on]
	tx-tcp6-segmentation: off [requested on]
net.ipv4.tcp_retries2 = 4
```

you acan apply the following manisfesto just TAG check release you wan in my case `<TAG>` is v0.12.0 aka TAG=`v0.12.0`

`kubectl apply -f https://github.com/vmware-tanzu/antrea/releases/download/v0.12.0/antrea-kind.yml`

output sample:

```shell
customresourcedefinition.apiextensions.k8s.io/antreaagentinfos.clusterinformation.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/antreacontrollerinfos.clusterinformation.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/clusternetworkpolicies.security.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/externalentities.core.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.security.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/tiers.security.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/traceflows.ops.antrea.tanzu.vmware.com created
serviceaccount/antctl created
serviceaccount/antrea-agent created
serviceaccount/antrea-controller created
clusterrole.rbac.authorization.k8s.io/aggregate-antrea-policies-edit created
clusterrole.rbac.authorization.k8s.io/aggregate-antrea-policies-view created
clusterrole.rbac.authorization.k8s.io/aggregate-traceflows-edit created
clusterrole.rbac.authorization.k8s.io/aggregate-traceflows-view created
clusterrole.rbac.authorization.k8s.io/antctl created
clusterrole.rbac.authorization.k8s.io/antrea-agent created
clusterrole.rbac.authorization.k8s.io/antrea-controller created
clusterrolebinding.rbac.authorization.k8s.io/antctl created
clusterrolebinding.rbac.authorization.k8s.io/antrea-agent created
clusterrolebinding.rbac.authorization.k8s.io/antrea-controller created
configmap/antrea-ca created
configmap/antrea-config-4h2tb7btgk created
service/antrea created
deployment.apps/antrea-controller created
apiservice.apiregistration.k8s.io/v1alpha1.stats.antrea.tanzu.vmware.com created
apiservice.apiregistration.k8s.io/v1beta1.controlplane.antrea.tanzu.vmware.com created
apiservice.apiregistration.k8s.io/v1beta1.networking.antrea.tanzu.vmware.com created
apiservice.apiregistration.k8s.io/v1beta1.system.antrea.tanzu.vmware.com created
apiservice.apiregistration.k8s.io/v1beta2.controlplane.antrea.tanzu.vmware.com created
daemonset.apps/antrea-agent created
mutatingwebhookconfiguration.admissionregistration.k8s.io/crdmutator.antrea.tanzu.vmware.com created
validatingwebhookconfiguration.admissionregistration.k8s.io/crdvalidator.antrea.tanzu.vmware.com created

```
check the agent and Ccontroller from `Antrea`

`kubectl get pods -n kube-system | grep antrea`

```shell
antrea-agent-5ts7b                              2/2     Running   0          14m
antrea-agent-6hklz                              2/2     Running   0          14m
antrea-agent-8xtb8                              2/2     Running   0          14m
antrea-agent-wksqp                              2/2     Running   0          14m
antrea-controller-94889cf46-9whh8               1/1     Running   0          14m
```
also check that coredns pods will change from `Pending` to `Running` state due to plumbing to communications between them

#### go some deep on Antrea

***i):from the output of this command :

`kubectl get all -n kube-system` 

```shell
NAME                                                READY   STATUS    RESTARTS   AGE
pod/antrea-agent-5ts7b                              2/2     Running   0          15m
pod/antrea-agent-6hklz                              2/2     Running   0          15m
pod/antrea-agent-8xtb8                              2/2     Running   0          15m
pod/antrea-agent-wksqp                              2/2     Running   0          15m
pod/antrea-controller-94889cf46-9whh8               1/1     Running   0          15m
pod/coredns-f9fd979d6-7bbs5                         1/1     Running   0          23m
pod/coredns-f9fd979d6-r9lz4                         1/1     Running   0          23m
pod/etcd-ernesto-control-plane                      1/1     Running   0          23m
pod/kube-apiserver-ernesto-control-plane            1/1     Running   0          23m
pod/kube-controller-manager-ernesto-control-plane   1/1     Running   0          23m
pod/kube-proxy-5ttl4                                1/1     Running   0          22m
pod/kube-proxy-pqsgg                                1/1     Running   0          22m
pod/kube-proxy-r2t6j                                1/1     Running   0          23m
pod/kube-proxy-vk9qd                                1/1     Running   0          22m
pod/kube-scheduler-ernesto-control-plane            1/1     Running   0          23m

NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
service/antrea     ClusterIP   10.96.83.166   <none>        443/TCP                  15m
service/kube-dns   ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP,9153/TCP   23m

NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/antrea-agent   4         4         4       4            4           kubernetes.io/os=linux   15m
daemonset.apps/kube-proxy     4         4         4       4            4           kubernetes.io/os=linux   23m

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/antrea-controller   1/1     1            1           15m
deployment.apps/coredns             2/2     2            2           23m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/antrea-controller-94889cf46   1         1         1       15m
replicaset.apps/coredns-f9fd979d6             2         2         2       23m
```

***ii):select one of the `antrea-agent-xxxxx` pod 

`antrea-agent-5ts7b`

***iii):and run this command:

`kubectl exec -n kube-system -it pod/antrea-agent-5ts7b -- ls /`

output sample:

```shell
Defaulting container name to antrea-agent.
Use 'kubectl describe pod/antrea-agent-5ts7b -n kube-system' to see all of the containers in this pod.
bin   dev  home  lib	lib64	media  opt   root  sbin  sys  usr
boot  etc  host  lib32	libx32	mnt    proc  run   srv	 tmp  var

```
***iv):enter in shell of agent to play with `OVS` with this command:

`kubectl exec -n kube-system -it pod/antrea-agent-5ts7b -- /bin/bash`

sample output:

```shell
Defaulting container name to antrea-agent.
Use 'kubectl describe pod/antrea-agent-5ts7b -n kube-system' to see all of the containers in this pod.
root@ernesto-worker3:/# antrea-agent --help
The Antrea agent runs on each node.

Usage:
  antrea-agent [flags]

Flags:
      --add_dir_header                   If true, adds the file directory to the header
      --alsologtostderr                  log to standard error as well as files
      --config string                    The path to the configuration file
  -h, --help                             help for antrea-agent
      --log-flush-frequency duration     Maximum number of seconds between log flushes (default 5s)
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --log_file string                  If non-empty, use this log file
      --log_file_max_num uint16          Maximum number of log files per severity level to be kept. Value 0 means unlimited.
      --log_file_max_size uint           Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)
      --logtostderr                      log to standard error instead of files (default true)
      --skip_headers                     If true, avoid header prefixes in the log messages
      --skip_log_headers                 If true, avoid headers when opening log files
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          number for the log level verbosity
      --version                          version for antrea-agent
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```

***v): run this command `ovs-<TAB>` to check all options of ovs commands

output example
```shell
ovs-appctl           ovs-docker           ovs-dpctl-top        ovs-parse-backtrace  ovs-pki              ovs-tcpundump        ovs-vsctl            
ovs-bugtool          ovs-dpctl            ovs-ofctl            ovs-pcap             ovs-tcpdump          ovs-vlan-test        ovs-vswitchd  

```
***vi): show `Bridge`

`ovs-vsctl show`
or from terminal we can directly exceute teh command in the container with this command:

`kubectl exec -n kube-system -it pod/antrea-agent-b5kpf -c antrea-agent ovs-vsctl show`

output example:

```shell
341952bb-9d54-4afe-8575-d190d9c46f04
    Bridge br-phy
        fail_mode: standalone
        datapath_type: netdev
        Port eth0
            Interface eth0
        Port br-phy
            Interface br-phy
                type: internal
    Bridge br-int
        datapath_type: netdev
        Port antrea-tun0
            Interface antrea-tun0
                type: geneve
                options: {csum="true", key=flow, remote_ip=flow}
        Port antrea-gw0
            Interface antrea-gw0
                type: internal
    ovs_version: "2.14.0"
```

***vii): list all ports on OVS `Bridge`

`ovs-vsctl list-ports br-int`

output example:

```shell
antrea-gw0
antrea-tun0
coredns--975ed0
root@antreakind-control-plane:/# ovs-vsctl list-br
br-int
br-phy
root@antreakind-control-plane:/# ovs-vsctl list-ports br-int
antrea-gw0
antrea-tun0
coredns--975ed0
root@antreakind-control-plane:/# ovs-vsctl list-ifaces br-int
antrea-gw0
antrea-tun0
coredns--975ed0
```
------------------------------
##### short list of what I used during my exploration and tests
-----------------------------
```shell
ovs-ofctl show br-int
ovs-ofctl dump-ports br-int
ovs-vsctl list-br
ovs-vsctl list-ports br-int
ovs-vsctl list-ifaces br-int
ovs-ofctl dump-flows br-int
ovs-ofctl snoop br-int
ovs-vsctl add-br br-int - set bridge br-int datapath_type=geneve
ovs-vsctl del-br br-int
ovs-vsctl del-controller br-int
ovs-vsctl set Bridge br-int stp_enable=true
ovs-vsctl add-port br-iny ge-1/1/1 type=pronto options:link_speed=1G
ovs-vsctl del-port br-int ge-1/1/
ovs-ofctl add-flow br-int in_port=1,actions=output:2
ovs-ofctl mod-flows br-int in_port=1,dl_type=0x0800,nw_src=100.10.0.1,actions=output:2
ovs-ofctl add-flow br-int in_port=1,actions=output:2,3,4
ovs-ofctl add-flow br-int in_port=1,actions=output:4
ovs-ofctl del-flows br-int
ovs-ofctl mod-port br-int 1 no-flood
ovs-ofctl add-flow br-int in_port=1,dl_type=0x0800,nw_src=192.168.1.241,actions=output:3
ovs-ofctl add-flow br-int in_port=4,dl_type=0x0800,dl_src=60:eb:69:d2:9c:dd,nw_src=198.168.0.2,nw_dst=192.168.0.55,actions=output:1
ovs-ofctl mod-flows br-int in_port=4,dl_type=0x0800,nw_src=192.168.0.45,actions=output:3
ovs-ofctl del-flows br-int in_port=1
```
-------------------------------------------

***viii): in addition to list we can explore the binding with Pod networking with `OVS Bridge`

`ip -c a`

example output
```shell
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN group default qlen 1000
    link/tunnel6 :: brd ::
4: ovs-netdev: <BROADCAST,MULTICAST,PROMISC> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether d2:db:53:ae:4e:81 brd ff:ff:ff:ff:ff:ff
5: br-phy: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 scope global br-phy
       valid_lft forever preferred_lft forever
6: antrea-gw0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 32:68:3a:5a:d5:44 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.1/24 brd 192.168.2.255 scope global antrea-gw0
       valid_lft forever preferred_lft forever
15: eth0@if16: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

**Step 5:** Install `Octant` and install  the `Antrea Octant` plugin

Install Octant on Mac

`brew install octant`

Clone Octant git repo,

`git clone https://github.com/vmware-tanzu/octant.git`

plugins included for test
Build an `Octant` test plugin `https://reference.octant.dev/?path=/docs/docs-plugins-2-go-plugins--page`

`cd octant`

`go run build.go install-test-plugin`

sample output

```shell
2021/03/05 00:59:13 Plugin path: /Users/rcastaneda/.config/octant/plugins
2021/03/05 00:59:13 Running: /usr/local/bin/go build -o /Users/rcastaneda/.config/octant/plugins/octant-sample-plugin github.com/vmware-tanzu/octant/cmd/octant-sample-plugin
```

verify installation of test plugin


`ls $HOME/.config/octant/plugin/`

```
octant-sample-plugin
```

#### `Antrea Octant` Plugin
apply from recent clone `Antrea` repo as well, this will be useful for the `Antrea Octant` Plugin or get from this link

`https://github.com/vmware-tanzu/antrea/releases/download/v0.12.2/antrea-octant-plugin-darwin-x86_64`

so then move to the default octant plpugin folder:

`mv antrea-octant-plugin-darwin-x86_64 ~/.config/octant/plugins/`

export kubeconf

`export KUBECONFIG=~/.kube/config`

run `Octant` from terminal

output example
```shell
`Octant`

2021-03-05T01:07:13.876-0600	INFO	dash/watcher.go:116	watching config file	{"component": "config-watcher", "config": "/Users/rcastaneda/.kube/config"}
2021-03-05T01:07:13.879-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/cronJob", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/suspendCronJob", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/resumeCronJob", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/containerEditor", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "overview/startPortForward", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "overview/stopPortForward", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/cordon", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/apply", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/deploymentConfiguration", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/serviceEditor", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/uncordon", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/update", "module-name": "overview"}
2021-03-05T01:07:13.880-0600	INFO	module/manager.go:83	registering action	{"component": "module-manager", "actionPath": "action.octant.dev/deleteObject", "module-name": "configuration"}
2021-03-05T01:07:13.882-0600	INFO	plugin/manager.go:372	watching for new JavaScript plugins in ["/Users/rcastaneda/.config/octant/plugins"]
2021-03-05T01:07:14.312-0600	INFO	plugin/manager.go:634	registering plugin action	{"plugin-name": "antrea-octant-plugin-darwin-x86_64", "action-path": "traceflow/addTf"}
2021-03-05T01:07:14.313-0600	INFO	plugin/manager.go:634	registering plugin action	{"plugin-name": "antrea-octant-plugin-darwin-x86_64", "action-path": "traceflow/showGraphAction"}
2021-03-05T01:07:14.313-0600	INFO	plugin/manager.go:647	registered plugin "antrea-octant-plugin"	{"plugin-name": "antrea-octant-plugin-darwin-x86_64", "cmd": "/Users/rcastaneda/.config/octant/plugins/antrea-octant-plugin-darwin-x86_64", "metadata": {"Name":"antrea-octant-plugin","Description":"Antrea","Capabilities":{"IsModule":true,"ActionNames":["traceflow/addTf","traceflow/showGraphAction"]}}}
2021-03-05T01:07:14.313-0600	INFO	plugin/manager.go:655	plugin supports navigation	{"plugin-name": "antrea-octant-plugin-darwin-x86_64"}
2021-03-05T01:07:14.725-0600	INFO	plugin/manager.go:634	registering plugin action	{"plugin-name": "octant-sample-plugin", "action-path": "action.octant.dev/example"}
2021-03-05T01:07:14.725-0600	INFO	plugin/manager.go:647	registered plugin "plugin-name"	{"plugin-name": "octant-sample-plugin", "cmd": "/Users/rcastaneda/.config/octant/plugins/octant-sample-plugin", "metadata": {"Name":"plugin-name","Description":"a description","Capabilities":{"SupportsPrinterConfig":[{"Group":"","Version":"v1","Kind":"Pod"}],"SupportsTab":[{"Group":"","Version":"v1","Kind":"Pod"}],"IsModule":true,"ActionNames":["action.octant.dev/example"]}}}
2021-03-05T01:07:14.725-0600	INFO	plugin/manager.go:655	plugin supports navigation	{"plugin-name": "octant-sample-plugin"}
2021-03-05T01:07:16.971-0600	INFO	dash/dash.go:596	using api service
2021-03-05T01:07:16.971-0600	INFO	dash/dash.go:505	Dashboard is available at http://127.0.0.1:7777
```
this will open in auto a web page on your browser 

![image](https://user-images.githubusercontent.com/5790758/110080028-890ab200-7d4f-11eb-875c-8c0501f72885.png)




==========Optional k9s UI in terminal==========
snap install k9s  # if you already using ver. 20 of ubuntu as in my case snap comes as default

check by invoke k9s from cli, if fails check doing this export for config file:

`export KUBECONFIG=$HOME/.kube/config`

other issue is the permission on .k9s: permissions denied try this:

`mkdir ~/.k9s`

============================================

**Step 6:** Install [MetalLB](https://metallb.universe.tf/) this works as a charm in Linux
**** since this is your sand box you can work with `NodePort` service for your local Pods to expose them, sometimes for Security paranoid reasons the process to make exposure of `docker network` on MAc will force to use some "hacks" to make it work, and will be a challenge to make it work with those restrictions ****
`https://kind.sigs.k8s.io/docs/user/loadbalancer/`

```shell
git:(master) ✗  kubectl explain service.spec.externalTrafficPolicy
KIND:     Service
VERSION:  v1

FIELD:    externalTrafficPolicy <string>

DESCRIPTION:
     externalTrafficPolicy denotes if this Service desires to route external
     traffic to node-local or cluster-wide endpoints. "Local" preserves the
     client source IP and avoids a second hop for LoadBalancer and Nodeport type
     services, but risks potentially imbalanced traffic spreading. "Cluster"
     obscures the client source IP and may cause a second hop to another node,
     but should have good overall load-spreading.
```



Create namespace:

`kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml`

output example:
```
namespace/metallb-system created
```
deploy from manifest:

`kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml`

output example
```shell
podsecuritypolicy.policy/controller created
podsecuritypolicy.policy/speaker created
serviceaccount/controller created
serviceaccount/speaker created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
role.rbac.authorization.k8s.io/config-watcher created
role.rbac.authorization.k8s.io/pod-lister created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/config-watcher created
rolebinding.rbac.authorization.k8s.io/pod-lister created
daemonset.apps/speaker created
deployment.apps/controller created
```
check pod creation with this command:

`kubectl -n metallb-system get pods`

output example:

```shell

NAME                              READY   STATUS                       RESTARTS   AGE
pod/controller-65db86ddc6-5zqrf   1/1     Running                      0          57s
pod/speaker-8cjtq                 0/1     CreateContainerConfigError   0          57s
pod/speaker-9gmzm                 0/1     CreateContainerConfigError   0          57s

```

create secret for MetalLB:

`kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"`

then check again statu of Metal LB pods

example output

```shell
kubectl -n metallb-system get pods

NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-65db86ddc6-5zqrf   1/1     Running   0          2m24s
pod/speaker-8cjtq                 1/1     Running   0          2m24s
pod/speaker-9gmzm                 1/1     Running   0          2m24s
```

**Step 7:** Find out the IP addresses being used by the nodes in the K8S cluster:

`kubectl get nodes -o wide`

You'll get output similar to the following:

```shell
NAME                 STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                     KERNEL-VERSION      CONTAINER-RUNTIME
ernesto-control-plane   Ready    master   5m23s   v1.19.1   172.18.0.4    <none>        Ubuntu Groovy Gorilla (development branch)   4.4.0-185-generic   containerd://1.4.0
ernesto-worker          Ready    <none>   4m42s   v1.19.1   172.18.0.3    <none>        Ubuntu Groovy Gorilla (development branch)   4.4.0-185-generic   containerd://1.4.0
ernesto-worker2         Ready    <none>   4m41s   v1.19.1   172.18.0.2    <none>        Ubuntu Groovy Gorilla (development branch)   4.4.0-185-generic   containerd://1.4.0

```
**Step 8:** Find the [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) of the bridge into tthe cluster

`ip a s`

You'll get output similar to the following

```shell
.
.
.
4: br-956cdadd3d33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:8b:33:c3:fb brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.19.255.255 scope global br-956cdadd3d33
       valid_lft forever preferred_lft forever
    inet6 fc00:f853:ccd:e793::1/64 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::42:8bff:fe33:c3fb/64 scope link
       valid_lft forever preferred_lft forever
    inet6 fe80::1/64 scope link
       valid_lft forever preferred_lft forever
.
.
.
```
using `sipcalc` we can have the range from CIDR to use for the LB depoending on your netwokr settings your will enter differten netwokr segmetn in this example I use `172.18.0.1/16`:

`sipcalc 172.18.0.1/16`

output example:

```shell
-[ipv4 : 172.18.0.1/16] - 0

[CIDR]
Host address		- 172.18.0.1
Host address (decimal)	- 2886860801
Host address (hex)	- AC120001
Network address		- 172.18.0.0
Network mask		- 255.255.0.0
Network mask (bits)	- 16
Network mask (hex)	- FFFF0000
Broadcast address	- 172.18.255.255
Cisco wildcard		- 0.0.255.255
Addresses in network	- 65536
Network range		- 172.18.0.0 - 172.18.255.255
Usable range		- 172.18.0.1 - 172.18.255.254
```

**Step 12:** Create the ConfigMap for MetalLB using `vim` getting the range of addresses to include from the values provided in the field, `Usable range` from the output of the previous step.

`vi metallb-config.yaml`

Add the following content to `metallb-config.yaml`

```yaml

apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2 #we are not going to peer at L3 at this point
      addresses:
      - 172.18.255.1-172.18.255.250

```

**WHERE**

`172.18.255.1-172.18.255.250` are the IP addreses determined from the steps above.

**Step 13:** Apply the ConfigMap to the K8S cluster

`kubectl apply -f metallb-config.yaml`

**Step 14:** Create a test K8S deployment and bind it to a service

`kubectl create deploy nginx --image nginx`

output example:

```shell
NAME                         READY   STATUS              RESTARTS   AGE
pod/nginx-6799fc88d8-zsmdt   0/1     ContainerCreating   0          2s
```
then expose teh test app:

`kubectl expose deploy nginx --port 80 --type LoadBalancer`
```
service/nginx exposed
```
chekc if this is ok:

`kubectl get all`

output example:

```shell
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-zsmdt   1/1     Running   0          91s

NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
service/kubernetes   ClusterIP      10.96.0.1      <none>         443/TCP        7h45m
service/nginx        LoadBalancer   10.96.75.187   172.18.255.1   80:31725/TCP   4s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           91s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6799fc88d8   1         1         1       91s

```

**Step 15:** Confirm that the `nginx` service is bound to an external IP address

`kubectl get services`

You'll get out put similar to the following:

```shell
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
service/kubernetes   ClusterIP      10.96.0.1      <none>         443/TCP        7h45m
service/nginx        LoadBalancer   10.96.75.187   172.18.255.1   80:31725/TCP   4s
```
Notice that the nginx service publishes an `EXTERNAL-IP`.

**Step 16:** Run the `curl` against the `EXTERNAL-IP`, for example:

`curl http://172.18.255.1`

You'll get output similar to the following:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
Cleanup
`kubectl delete deploy nginx` 
`kubectl delete service nginx`

**Step 16:** Install [Helm](https://helm.sh/)

**Step 17:** Install [Kubeapps]

deploy wordpress from kubeapps using helm

**Step 18:** Install [ARgoCD] and [Concourse]

test with nginx from repo

**Step 19:** Install robot app

**Step 21:** install [TMC]

test netwokr poly deny all and check the trace from antrea plugin on octant

**Step 22:** Install [TO]

do a tracing?

**Step 23:** Install [TSM]

create a global NS and do a GLB between two kind clusters



https://courses.academy.tigera.io/


We will be creating 4 VMs (3 Kubernetes nodes and 1 standalone host) each with predefined static IP addresses:

Control (198.19.0.1) - Kubernetes control-plane node
Node1 (198.19.0.2) - Kubernetes worker node
Node2 (198.19.0.3) - Kubernetes worker node
Host1 (198.19.15.254) - General purpose host




cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)





```bash

wget https://raw.githubusercontent.com/marcredhat/calicolab/main/installsnapcraft.sh
wget https://raw.githubusercontent.com/marcredhat/calicolab/main/installmultipass.sh
wget https://raw.githubusercontent.com/marcredhat/calicolab/main/installk8s.sh

chmod +x ./installsnapcraft.sh
chmod +x ./installmultipass.sh
chmod +x ./installk8s.sh

./installsnapcraft.sh
./installmultipass.sh
./installk8s.sh
```

```bash
Expected result:
Name                    State             IPv4             Image
control                 Running           10.40.99.90      Ubuntu 20.04 LTS
host1                   Running           10.40.99.68      Ubuntu 20.04 LTS
node1                   Running           10.40.99.161     Ubuntu 20.04 LTS
node2                   Running           10.40.99.61      Ubuntu 20.04 LTS
```

```bash
multipass shell host1
```

```bash
ubuntu@host1:~$ kubectl get nodes -A
```

Expected result:
```text
NAME      STATUS     ROLES    AGE     VERSION
node2     NotReady   <none>   3m56s   v1.18.10+k3s1
control   NotReady   master   6m39s   v1.18.10+k3s1
node1     NotReady   <none>   5m8s    v1.18.10+k3s1
```

```bash
kubectl create -f https://docs.projectcalico.org/archive/v3.16/manifests/tigera-operator.yaml
kubectl get pods -n tigera-operator
```

Expected result:
```text
kubectl create -f https://docs.projectcalico.org/archive/v3.16/manifests/tigera-operator.yaml
namespace/tigera-operator created
podsecuritypolicy.policy/tigera-operator created
serviceaccount/tigera-operator created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
deployment.apps/tigera-operator created
ubuntu@host1:~$ kubectl get pods -n tigera-operator
NAME                               READY   STATUS              RESTARTS   AGE
tigera-operator-55769bd645-fsmds   0/1     ContainerCreating   0          6s
```

```bash
kubectl get pods -n tigera-operator
NAME                               READY   STATUS    RESTARTS   AGE
tigera-operator-55769bd645-fsmds   1/1     Running   0          55s
```

```bash
cat <<EOF | kubectl apply -f -
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    containerIPForwarding: Enabled
    ipPools:
    - cidr: 198.19.16.0/21
      natOutgoing: Enabled
      encapsulation: None
EOF
```

```
kubectl get tigerastatus/calico
The output from the command when the installation is complete is:

NAME     AVAILABLE   PROGRESSING   DEGRADED   SINCE
calico   True        False         False      10m
We can review the environment now by invoking:

kubectl get pods -A
Example output:

NAMESPACE         NAME                                      READY   STATUS      RESTARTS   AGE
tigera-operator   tigera-operator-84c5c5d6df-zb49b          1/1     Running     0          5m48s
calico-system     calico-typha-868bb997ff-l22n7             1/1     Running     0          4m6s
calico-system     calico-typha-868bb997ff-fvmws             1/1     Running     0          3m24s
calico-system     calico-typha-868bb997ff-8qt45             1/1     Running     0          3m24s
calico-system     calico-node-r94mp                         1/1     Running     0          4m6s
calico-system     calico-node-w5ptt                         1/1     Running     0          4m6s
calico-system     calico-node-zgrvb                         1/1     Running     0          4m6s
kube-system       helm-install-traefik-t68vd                0/1     Completed   0          35m
kube-system       metrics-server-7566d596c8-pccvz           1/1     Running     2          35m
kube-system       local-path-provisioner-6d59f47c7-gh97b    1/1     Running     2          35m
calico-system     calico-kube-controllers-89cf65556-c7gz7   1/1     Running     3          4m6s
kube-system       coredns-7944c66d8d-f4q6g                  1/1     Running     0          35m
kube-system       svclb-traefik-9bxg2                       2/2     Running     0          32s
kube-system       svclb-traefik-pb72f                       2/2     Running     0          32s
kube-system       svclb-traefik-l6mzn                       2/2     Running     0          32s
kube-system       traefik-758cd5fc85-8hcdx                  1/1     Running     0          32s
Reviewing Calico pods
Let's take a look at the Calico pods that have been installed by the operator.

kubectl get pods -n calico-system
Example Output:

NAME                                       READY   STATUS    RESTARTS   AGE
calico-typha-5d788c654b-56wp9              1/1     Running   0          4h28m
calico-node-2bkv5                          1/1     Running   0          4h28m
calico-kube-controllers-5dcfdbc5f4-vpgx5   1/1     Running   0          4h28m
calico-node-8465h                          1/1     Running   0          4h26m
calico-typha-5d788c654b-wn7gf              1/1     Running   0          4h24m
calico-node-qmq5j                          1/1     Running   0          3h57m
calico-typha-5d788c654b-rd8kl              1/1     Running   0          3h56m
From here we can see that there are different pods that are deployed.

calico-node: Calico-node runs on every Kubernetes cluster node as a DaemonSet. It is responsible for enforcing network policy, setting up routes on the nodes, plus managing any virtual interfaces for IPIP, VXLAN, or WireGuard.
calico-typha: Typha is as a stateful proxy for the Kubernetes API server. It's used by every calico-node pod to query and watch Kubernetes resources without putting excessive load on the Kubernetes API server.  The Tigera Operator automatically scales the number of Typha instances as the cluster size grows.
calico-kube-controllers: Runs a variety of Calico specific controllers that automate synchronization of resources. For example, when a Kubernetes node is deleted, it tidies up any IP addresses or other Calico resources associated with the node.
Reviewing Node Health
Finally, we can review the health of our Kubernetes nodes by invoking the kubectl command.

kubectl get nodes -A
Example output:

NAME      STATUS   ROLES    AGE   VERSION
control   Ready    master   37m   v1.18.10+k3s1
node2     Ready    <none>   16m   v1.18.10+k3s1
node1     Ready    <none>   31m   v1.18.10+k3s1
Now we can see that our Kubernetes nodes have a status of Ready and are operational. Calico is now installed on your cluster and you may proceed to the next module: Installing the Sample Application.



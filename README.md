
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
control                 Running           10.90.207.88     Ubuntu 20.04 LTS
host1                   Running           10.90.207.225    Ubuntu 20.04 LTS
node1                   Running           10.90.207.205    Ubuntu 20.04 LTS
node2                   Running           10.90.207.96     Ubuntu 20.04 LTS
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

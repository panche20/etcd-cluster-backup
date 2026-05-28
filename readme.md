# ETCD Backup & Restore

### Install ETCDCTL utility. 

**etcdctl is the client utility for interacting with a live etcd cluster.**

```
ETCD_VER=v3.6.11
curl -LO https://github.com/etcd-io/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf etcd-${ETCD_VER}-linux-amd64.tar.gz
sudo mv etcd-${ETCD_VER}-linux-amd64/etcdctl /usr/local/bin/

# Verify etcdctl version:
etcdctl version
```

### Install ETCDUTL utility. 

**etcdutl is the administrative utility used for offline operations like snapshot restore and database inspection.**

```
ETCD_VER=v3.6.1
curl -LO https://github.com/etcd-io/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf etcd-${ETCD_VER}-linux-amd64.tar.gz
sudo mv etcd-${ETCD_VER}-linux-amd64/etcdutl /usr/local/bin/
```

## ETCD BACKUP

### Step 1 : Verify etcd POD

```
kubectl get pods -n kube-system | grep etcd
```

### Step 2 : Locate etcd Certificates

```
# kubeadm stores certs here:

/etc/kubernetes/pki/etcd/

# Verify:
sudo ls /etc/kubernetes/pki/etcd

# You should see :
- ca.crt
- server.crt
- server.key
```

### Step 3 : Create Snapshot

```
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save snapshot.db
```

### Step 4 : Verify Snapshot

```
ls -lh snapshot.db
```

## ETCD RESTORE

### Step 5 : Stop Kubelet

```
sudo systemctl stop kubelet
```

### Step 6 : Backup Existing etcd Data

```
sudo mv /var/lib/etcd /var/lib/etcd-backup
```

### Step 7 : Restore Snapshot

```
sudo ETCDCTL_API=3 etcdutl snapshot restore snapshot.db \
--data-dir=/var/lib/etcd
```

### Step 8 : Verify Static Pod Manifest

```
sudo cat /etc/kubernetes/manifests/etcd.yaml

Verify:
--data-dir=/var/lib/etcd
```

### Step 9 : Restart kubelet

```
sudo systemctl start kubelet
```

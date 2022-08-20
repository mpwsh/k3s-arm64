### OS Dependencies

In *all* nodes:
```bash
sudo apt install -y nfs-common  open-iscsi
```

### Basic auth
htpasswd -c auth username
kubectl create secret generic basic-auth --from-file=auth
kubectl get secret basic-auth -o yaml

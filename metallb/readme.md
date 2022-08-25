# MetalLB

### Using BGP
MetalLB docs explain things pretty well, but found [this great article](https://medium.com/@ipuustin/using-metallb-as-kubernetes-load-balancer-with-ubiquiti-edgerouter-7ff680e9dca3) from Ismo Puustinen which covers the Ubiquiti Edgerouter specifically, which is the one used in my setup.


### Deploy MetalLB
For more info check: MetalLB [installation documentation](https://metallb.universe.tf/installation/)

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.4/config/manifests/metallb-native.yaml
```

Modify `bgp-peer.yaml`, `ip-pool.yaml` and `bgp-advertisement.yaml` to suit your needs and apply.
```bash
kubectl apply -f bgp-peer.yaml -f ip-pool.yaml -f bgp-advertisement.yaml
```

### Configuring router.
SSH into the router and run.
```bash
configure
set protocols bgp 64512 parameters router-id 192.168.1.1
set protocols bgp 64512 neighbor 192.168.1.8 remote-as 64512
set protocols bgp 64512 neighbor 192.168.1.9 remote-as 64512
set protocols bgp 64512 neighbor 192.168.1.10 remote-as 64512
set protocols bgp 64512 neighbor 192.168.1.11 remote-as 64512
set protocols bgp 64512 maximum-paths ibgp 32
commit
save
exit
```

### Validation
```bash
show ip bgp neighbors
```


### Testing
Deploy a service with `type: LoadBalancer`, you can use [redis](../redis) as an example. You should get an IP from the pool assigned as `External-IP`

### output
```bash
~> kubectl get svc -n redis
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
redis-server   LoadBalancer   10.43.121.246   192.168.1.210   6379:31895/TCP   51m
```

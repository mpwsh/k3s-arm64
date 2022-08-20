### Dependencies
 - [MetalLB](../metallb)

Default manifest for BareMetal deployment, from [kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx/).


Deploy the manifest from the repository
```bash
kubectl apply -f deploy.yaml
```

Grab and deploy the latest with:
```bash
curl -sfL https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml | sed 's#type: NodePort#type: LoadBalancer#g' | kubectl apply -f -
```

### Dependencies
 - [NGINX Ingress](../nginx-ingress)
 - [Cert Manager](../cert-manager) (with cluster Issuer)
 - [External DNS](../external-dns) (optional)

### How to deploy
Find and replace all instances of the domain `mpw.sh` and replace with your own.

This can be found at:
- Value of annotation `nginx.ingress.kubernetes.io/server-snippet`, `tls.host` and `spec.rules.host` in [ingress.yaml](ingress.yaml)
- `server_name` in [configmap.yaml](configmap.yaml).

Modify [pvc.yaml](pvc.yaml) and set-up the desired storage size.

[Conduit](https://conduit.rs) configuration resides in [configmap.yaml](configmap.yaml) manifest.
You can keep the defaults untouched, but feel free to modify the config.

After you're done, apply all the manifests with `kubectl`.


```bash
#Create a namespace for Conduit
kubectl create ns conduit
#Deploy all the manifests
kubectl apply -f . -n conduit
```

### Testing
Check if the pod is running correctly
```bash
kubectl get pods -n conduit
```

Get logs from the server
```bash
kubectl logs -f deploy/conduit
```
If everyhing went well, you should be able to get the server name and running version

```bash
curl https://<your-domain>/_matrix/federation/v1/version
```

Testing `.well-known` client and server paths.
```bash
#client
https://<your-domain>/.well-known/matrix/client
#server
https://<your-domain>/.well-known/matrix/server
```

All endpoints should respond.

### Create secret with Github Personal access token

```bash
secret_name=controller-manager
kubectl create secret generic $secret_name \
--from-literal=github_token="<your-token>" \
--dry-run=client -o json | \
kubeseal --controller-namespace sealed-secrets --controller-name=sealed-secrets-controller > sealed-$secret_name.json
```

### Apply the secret

```bash
kubectl apply -f sealed-$secret_name.json
```

### TL;DR:
```bash
#Sealing a secret
secret_name=cloudflare-creds
 kubectl create secret generic $secret_name --dry-run=client --from-literal=token=<token> -o json \
 | kubeseal --controller-namespace sealed-secrets --controller-name=sealed-secrets-controller \
 > sealed-$secret_name.json
```

Instructions from [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets) repository.

### Usage
- [Usage](#usage)
  - [Managing existing secrets](#managing-existing-secrets)
  - [Update existing secrets](#update-existing-secrets)
  - [Raw mode (experimental)](#raw-mode-experimental)

## Usage

```bash
# Create a json/yaml-encoded Secret somehow:
# (note use of `--dry-run` - this is just a local file!)
echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json >mysecret.json

# This is the important bit:
# (note default format is json!)
kubeseal <mysecret.json >mysealedsecret.json
#kubeseal --controller-namespace sealed-secrets --controller-name=sealed-secrets-controller <mysecret.json >mysealedsecret.json
#Use this if kubeseal is in another namespace, or didnt deploy with FullNameOverride.
#kubeseal --controller-namespace tooling --controller-name=sealed-secrets -n tooling <mysecret.json >mysealedsecret.json

# mysealedsecret.json is safe to upload to github, post to twitter,
# etc.  Eventually:
kubectl create -f mysealedsecret.json

# Profit!
kubectl get secret mysecret
```

Note the `SealedSecret` and `Secret` must have **the same namespace and
name**. This is a feature to prevent other users on the same cluster
from re-using your sealed secrets. See the [Scopes](#scopes) section for more info.

`kubeseal` reads the namespace from the input secret, accepts an explicit `--namespace` arg, and uses
the `kubectl` default namespace (in that order). Any labels,
annotations, etc on the original `Secret` are preserved, but not
automatically reflected in the `SealedSecret`.

By design, this scheme *does not authenticate the user*.  In other
words, *anyone* can create a `SealedSecret` containing any `Secret`
they like (provided the namespace/name matches).  It is up to your
existing config management workflow, cluster RBAC rules, etc to ensure
that only the intended `SealedSecret` is uploaded to the cluster.  The
only change from existing Kubernetes is that the *contents* of the
`Secret` are now hidden while outside the cluster.

### Managing existing secrets

If you want `SealedSecret` controller to take management of an existing `Secret` (i.e. overwrite it when unsealing a `SealedSecret` with the same name and namespace), then you have to annotate that `Secret` with the annotation `sealedsecrets.bitnami.com/managed: "true"` ahead applying the [Usage](#usage) steps.

### Update existing secrets

If you want to add or update existing sealed secrets without having the cleartext for the other items,
you can just copy&paste the new encrypted data items and merge it into an existing sealed secret.

You must take care of sealing the updated items with a compatible name and namespace (see note about scopes above).

You can use the `--merge-into` command to update an existing sealed secrets if you don't want to copy&paste:

```bash
echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json \
  | kubeseal > mysealedsecret.json
echo -n baz | kubectl create secret generic mysecret --dry-run=client --from-file=bar=/dev/stdin -o json \
  | kubeseal --merge-into mysealedsecret.json
```

### Raw mode (experimental)

Creating temporary Secret with the `kubectl` command, only to throw it away once piped to `kubeseal` can
be a quite unfriendly user experience. We're working on an overhaul of the CLI experience. In the meantime,
we offer an alternative mode where kubeseal only cares about encrypting a value to stdout, and it's your responsibility to put it inside a `SealedSecret` resource (not unlike any of the other k8s resources).

It can also be useful as a building block for editor/IDE integrations.

The downside is that you have to be careful to be consistent with the sealing scope, the namespace and the name.

See [Scopes](#scopes)

`strict` scope (default):

```console
$ echo -n foo | kubeseal --raw --from-file=/dev/stdin --namespace bar --name mysecret
AgBChHUWLMx...
```

`namespace-wide` scope:

```console
$ echo -n foo | kubeseal --raw --from-file=/dev/stdin --namespace bar --scope namespace-wide
AgAbbFNkM54...
```
Include the `sealedsecrets.bitnami.com/namespace-wide` annotation in the `SealedSecret`
```yaml
metadata:
  annotations:
    sealedsecrets.bitnami.com/namespace-wide: "true"
```

`cluster-wide` scope:

```console
$ echo -n foo | kubeseal --raw --from-file=/dev/stdin --scope cluster-wide
AgAjLKpIYV+...
```
Include the `sealedsecrets.bitnami.com/cluster-wide` annotation in the `SealedSecret`
```yaml
metadata:
  annotations:
    sealedsecrets.bitnami.com/cluster-wide: "true"
```

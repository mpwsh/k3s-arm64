### How to use this repo
 - Install docker
 - Deploy a [k3s](https://k3s.io/) cluster
 - Install [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
 - Fork the repository
 - Find and replace all instances of `mpw.sh` domain in the ingresses and replace with your own
 - Deploy [ArgoCD](argocd)
 - Find and replace all instances of this repository in argoCD `apps` with your repository
 - Apply the `AppProject` manifest of the desired application
 - Apply the `Application` manifest of the desired application


### Deploying k3s on a rpi4o
Enable  legacy IP tables. [Rancher Docs](https://rancher.com/docs/k3s/latest/en/advanced/#enabling-legacy-iptables-on-raspberry-pi-os
)
```bash
sudo iptables -F
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo reboot
```

```bash
#Change the hostname of the node to something unique
vi /etc/hostname
#Kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl"
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
sudo mkdir -p /etc/rancher/k3s && \
cat <<EOF  | sudo tee -a /etc/rancher/k3s/config.yaml
kubelet-arg:
  - "kube-reserved=cpu=500m,memory=1Gi,ephemeral-storage=2Gi"
  - "system-reserved=cpu=500m, memory=1Gi,ephemeral-storage=2Gi"
  - "eviction-hard=memory.available<500Mi,nodefs.available<10%"
EOF

#change with your domain
domain=mpw.sh
#If you want to save your cluster data in a different path, add --data-dir /path/k3s
#Remove '--disable local-storage if you're not going to use -longhorn-
curl -sfL https://get.k3s.io | sh -s - server --disable traefik --disable servicelb --write-kubeconfig-mode 644 --tls-san k3s.$domain --https-listen-port 6443 --disable-cloud-controller --disable local-storage
#Move kubeconfig to your home path
mkdir -p ~/.kube && sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
#Add this line to your bash_profile to set permanently.
export KUBECONFIG=~/.kube/config
echo "export KUBECONFIG=~/.kube/config" >> ~/.bash_profile
#Retrieve the node-token (You'll need this to add more nodes to your cluster)
cat /var/lib/rancher/k3s/server/node-token
```

Add more nodes to the cluster with:
```bash
#Enable legacy iptables with the commands above before running this command
curl -sfL https://get.k3s.io | K3S_URL=https://<lan-ip-of-your-master-node>:6443 K3S_TOKEN=<your-token> sh -
```

### Nothing works. HELP
I understand that this repo is not really documented (sorry). You can get a lot of valuable information and assistance online. k3s + raspberry pi is a very popular combo with multiple communities.
The website [rpi4cluster.com](https://rpi4cluster.com/) is an awesome source of information to get a better understanding of the whole setup.

Feel free to open an issue if you get stuck though, i'll do my best to help.

### TODO
- Add action to update manifest versions.

Example:
```bash
latest=$(curl -s https://api.github.com/repos/cert-manager/cert-manager/releases/latest | jq -r .tarball_url | cut -d/ -f8 | sed "s/v//g")
echo $latest
```

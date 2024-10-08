# homelab

## Install k3s
k3s doc: https://docs.k3s.io/installation/configuration
```bash
curl -sfL https://get.k3s.io | sh
```

Install kubectl: https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/

Configure kubectl credential.
```bash
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chmod 644 ~/.kube/config
sudo chown USER:GROUP ~/.kube/config
```

Then you see system pods.
```bash
kubectl get pods -A
```

## Setup Cloudflare Tunnel
First, install cloudflared. c.f. https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/
Then log in.
```bash
cloudflared tunnel login
```

Create a tunnel.
```bash
cloudflared tunnel create --credentials-file ./credentials.json k3s-tunnel
```

Create a secret to hold the credentials.
```bash
kubectl create secret generic cloudflare-tunnel-credentials -n cloudflare --from-file=credentials.json=credentials.json
```

Add a DNS record: CNAME from subdomain to the <tunnelID>.cfargotunnel.com.

## Argo CD
Follow [Argo CD Getting Started document](https://argo-cd.readthedocs.io/en/stable/getting_started/#2-download-argo-cd-cli).

Install Argo CD CLI.
If you have Homebrew, run
```bash
brew install argocd
```

Install Argo CD via helm.
```bash
helm dependency build ./helm/argocd
helm install -f helm/argocd/values.yaml argocd ./helm/argocd/
```

Then complete access to Argo CD with this command.
```bash
argocd login --core
```

I adopt app of apps pattern for deploy convinience. Just apply the root application manifest.
```bash
kubectl apply -f argocd/apps.yaml
```

## Grafana Setup
Extract the initial password for "admin" user.
```bash
kubectl get secrets -n monitoring kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

## Nextcloud Setup
Create a secret for PostgreSQL.
```bash
kubectl create secret generic nextcloud-postgresql-creds -n nextcloud --from-literal=admin-password='' --from-literal=user-password=''
```

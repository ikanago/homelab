# k3s-infra

**THIS REPOSITORY IS PUBLIC!!!**
Argo CD need authentication to access a private repository, but there is no way to access a private repository without specific user's credential.

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

First, install Argo CD by the following commands decribed in [doc](https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd).
Notice this installs core components, so Web UI and SSO are disabled.
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml
```

Install Argo CD CLI.
If you have Homebrew, run
```bash
brew install argocd
```

Then complete access to Argo CD with this command.
```bash
argocd login --core
```

We can skip step 3 to 5 in the document as we installed Argo CD core components only.

I adopt app of apps pattern for deploy convinience. Just apply the root application manifest.
```bash
kubectl apply -f argocd/apps.yaml
```

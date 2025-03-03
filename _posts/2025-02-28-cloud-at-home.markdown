---
layout: post
title:  "Cloud @ home"
date:   2025-02-28 16:36:17 +0530
categories: home-lab raspberry-pi
author: "Jatin Katyal"
---

Cloud computing isn't just for massive data centers; you can bring the cloud home! In this post, I'll walk through setting up a small but powerful cloud environment using Raspberry Pi, K3s for Kubernetes, Longhorn for storage, and Cloudflare Tunnel for remote access.

## Requirements
To get started, I purchased a Raspberry Pi from Amazon: [here](https://amzn.in/d/h8QG3A8). You’ll also need:

- A microSD card (32GB or larger, preferably high endurance)
- A stable power supply for Raspberry Pi
- A good cooling solution (optional but recommended)
- An internet connection

## Setting up K3s
K3s is a lightweight Kubernetes distribution that runs well on Raspberry Pi. The goal is to have a unified API to deploy applications, making it easier to scale by adding more Raspberry Pis later.

### Install K3s
```sh
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
```

### Set Kubeconfig
To access the cluster remotely:
```sh
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(whoami):$(whoami) ~/.kube/config
export KUBECONFIG=~/.kube/config
```

### Multi-node Setup
For future expansion, install K3s agent on additional Raspberry Pis:
```sh
curl -sfL https://get.k3s.io | K3S_URL=https://<master-ip>:6443 K3S_TOKEN=<TOKEN> sh -
```

## Setting up Longhorn for Storage
Longhorn provides distributed block storage, ensuring that data persists across Raspberry Pi nodes.

### Install Longhorn
```sh
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml
```

### Verify Installation
```sh
kubectl get pods -n longhorn-system
```

### Access Longhorn UI
Forward the service to access the dashboard:
```sh
kubectl port-forward -n longhorn-system svc/longhorn-frontend 8080:80
```
Now, open `http://localhost:8080` in your browser.

## Setting up Cloudflare Tunnel
Since home IPs are dynamic, Cloudflare Tunnel allows secure remote access to our home cloud without exposing ports.

### Install Cloudflare Tunnel
1. Sign up on Cloudflare and add your domain.
2. Install `cloudflared`:
   ```sh
   curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64 -o cloudflared
   chmod +x cloudflared
   sudo mv cloudflared /usr/local/bin/
   ```
3. Authenticate with Cloudflare:
   ```sh
   cloudflared tunnel login
   ```
4. Create a tunnel and configure it:
   ```sh
   cloudflared tunnel create home-cloud
   cloudflared tunnel route dns home-cloud <subdomain>.example.com
   ```
5. Run the tunnel:
   ```sh
   cloudflared tunnel run home-cloud
   ```

## Conclusion
With this setup, we now have a self-hosted cloud running Kubernetes on Raspberry Pi, with Longhorn for storage and Cloudflare Tunnel for secure remote access. As a next step, we can deploy applications and explore adding more Raspberry Pis for better reliability.

## References
- [K3s Documentation](https://k3s.io/)
- [Longhorn Docs](https://longhorn.io/docs/)
- [Cloudflare Tunnel Guide](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)

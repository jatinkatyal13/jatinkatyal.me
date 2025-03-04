---
layout: post
title:  "Cloud @ home"
date:   2025-02-28 16:36:17 +0530
categories: home-lab raspberry-pi
author: "Jatin Katyal"
og:
   title: Cloud @ home
   image: /assets/images/home_lab.jpg

---

Cloud computing isn't just for massive data centers; you can bring the cloud home! With AI becoming more and more powerful each day, it's crucial to bring this change closer to the actual consumer. While we have Phones, TVs, Computers, Alexas and what not, but today is the time we can buy brains, artificial brains. This brings me to my post, to research on this I needed a home lab and here I am documenting every step of it.

![Home Lab](/assets/images/home_lab.jpg)

In this post, I'll walk through setting up a small but powerful cloud environment using Raspberry Pi, K3s for Kubernetes, Longhorn for storage, and Cloudflare Tunnel for remote access.

## Index
1. [Requirements](#requirements)
2. [Single-node K3s Setup](#single-node-k3s-setup)
3. [Multi-node Setup](#multi-node-setup)
4. [Possible Errors](#possible-errors)
5. [Set Kubeconfig](#set-kubeconfig)
6. [Mount external SSD/USB](#mount-external-ssdusb)
   - [Format and Mount External USB Drive](#format-and-mount-external-usb-drive)
7. [Setting up Longhorn for Storage](#setting-up-longhorn-for-storage)
   - [Install `open-iscsi`](#install-open-iscsi)
   - [Install Longhorn](#install-longhorn)
   - [Verify Installation](#verify-installation)
8. [Setting up Cloudflare Tunnel](#setting-up-cloudflare-tunnel)
   - [Install Cloudflare Tunnel](#install-cloudflare-tunnel)
9. [Conclusion](#conclusion)
10. [References](#references)

## Requirements
To get started, I purchased a Raspberry Pi from Amazon: [here](https://amzn.in/d/h8QG3A8). You’ll also need:

- A microSD card (32GB or larger, preferably high endurance)
- A stable power supply for Raspberry Pi
- A good cooling solution (optional but recommended)
- An internet connection

## Single-node K3s Setup
K3s is a lightweight Kubernetes distribution that runs well on Raspberry Pi. The goal is to have a unified API to deploy applications, making it easier to scale by adding more Raspberry Pis later.

```sh
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
```

## Multi-node Setup
To expand your local cluster further to connect more devices or just have redundancy, install K3s agent on additional Raspberry Pis:

First, fetch the K3S token from the master node:
```sh
sudo cat /var/lib/rancher/k3s/server/node-token
```

Then, run the following command on each additional Raspberry Pi:
```sh
curl -sfL https://get.k3s.io | K3S_URL=https://<master-ip>:6443 K3S_TOKEN=<TOKEN> sh -
```

Replace `<master-ip>` with the IP address of your master node and `<TOKEN>` with the token you fetched from the master node.

## Possible Errors
During installation K3S service might fail to boot, you can check the error using following
```
journalctl -xeu k3s-agent.service
```

you'll find the following error
```
Mar 04 20:23:56 raspberrypi k3s[2006]: time="2025-03-04T20:23:56+05:30" level=fatal msg="failed to find memory cgroup (v2)"
```

To resolve this issue I updated the `/boot/firmware/cmdline.txt` and appended following
```
... cgroup_enable=memory cgroup_memory=1 systemd.unified_cgroup_hierarchy=1
```

## Set Kubeconfig
To access the cluster remotely:
```sh
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(whoami):$(whoami) ~/.kube/config
export KUBECONFIG=~/.kube/config
```

## Mount external SSD/USB
SD cards are usually not fit for storage workloads, affecting performance and reliability. To mount an external drive connect it to your Raspberry Pi and use following steps

### Format and Mount External USB Drive
To use an external USB drive for storage, follow these steps to format it to ext4 and mount it to `/mnt/RPI_USB_1`.

1. **Identify the USB Drive**:
   Plug in the USB drive and identify it using `lsblk`:
   ```sh
   lsblk
   ```
   Look for your USB drive, typically named something like `/dev/sda`.

2. **Format the Drive to ext4**:
   Replace `/dev/sda1` with your actual USB drive identifier:
   ```sh
   sudo mkfs.ext4 /dev/sda1
   ```

3. **Create Mount Point**:
   Create a directory to mount the USB drive:
   ```sh
   sudo mkdir -p /mnt/RPI_USB_1
   ```

4. **Mount the Drive**:
   Mount the USB drive to the created directory:
   ```sh
   sudo mount /dev/sda1 /mnt/RPI_USB_1
   ```

5. **Update fstab for Persistent Mount**:
   To ensure the drive mounts automatically on boot, add it to `/etc/fstab`:
   ```sh
   echo '/dev/sda1 /mnt/RPI_USB_1 ext4 defaults 0 0' | sudo tee -a /etc/fstab
   ```

Now, the USB drive is formatted to ext4 and mounted to `/mnt/RPI_USB_1`, ready to be used by Longhorn as the default mount path.

## Setting up Longhorn for Storage
Longhorn provides a nice setup to just use k8s PVC and have distributed block storage, ensuring that data persists across Raspberry Pi nodes.

### Install `open-iscsi`
Longhorn requires `open-iscsi` to be installed on all the nodes of the cluster. Read [here](https://longhorn.io/docs/archives/0.8.0/install/requirements/)
```sh
sudo apt-get update && sudo apt-get install -y open-iscsi
```

### Install Longhorn
Following is the helmfile.yaml that you can install on your cluster
```yaml
repositories:
  - name: longhorn
    url: https://charts.longhorn.io

releases:
  - name: longhorn
    namespace: longhorn-system
    chart: longhorn/longhorn
    version: 1.8.0
    values:
      - values-longhorn.yaml
```
and the values file looks like
```yaml
defaultSettings:
  defaultDataPath: /mnt/RPI_USB_1
```

### Verify Installation
```sh
kubectl get pods -n longhorn-system
```

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

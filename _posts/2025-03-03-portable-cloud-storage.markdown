---
layout: post
title:  "Portable Cloud Storage"
date:   2025-03-03 19:13:00 +0530
categories: home-lab raspberry-pi
author: "Jatin Katyal"
categories: [Cloud, Kubernetes, RaspberryPi, self-hosting, ente.io]
---

With cloud storage becoming an integral part of our digital lives, I wanted to create a truly personal cloud—one that is elastic, portable, and abstracted from the user. My goal was simple: self-host Ente.io, a secure photo and file storage service, on a Raspberry Pi cluster with the following properties:

1. **Elasticity**: Adding capacity should be as easy as plugging in another Raspberry Pi.
2. **User Abstraction**: The setup should be simple and seamless.
3. **Portability**: I should be able to carry my cloud anywhere.

After experimenting with various approaches, I successfully achieved this using Kubernetes on Raspberry Pi, Helm, and a distributed storage solution. This post walks through my setup and key takeaways.

---

## Building the Setup

### Creating a Kubernetes Cluster
I had already set up a home cloud using Kubernetes on Raspberry Pi, which I documented in [this post](https://jatinkatyal.me/cloud/kubernetes/raspberrypi/self-hosting/2025/02/28/cloud-at-home.html). This provided a solid foundation for deploying Ente.io and its dependencies.

### Introducing Ente.io
Ente.io is a secure photo and file storage service designed to provide end-to-end encryption for your data. It ensures that only you have access to your files, with no third-party interference. Ente's architecture is built around a central server, known as the "musem," which handles authentication, storage, and synchronization across devices.

### How Ente Works

1. **End-to-End Encryption**: Ente encrypts your files on your device before they are uploaded to the cloud. This means that even if someone gains access to the storage server, they cannot read your files without the encryption keys, which are only available on your devices.
2. **Cross-Platform Sync**: Ente supports synchronization across multiple devices, including smartphones, tablets, and computers. This ensures that your files are always up-to-date and accessible from any device.
3. **Secure Sharing**: You can securely share files and folders with others using Ente. The shared files remain encrypted, and only the intended recipients can decrypt and access them.

Below is the architecture diagram of the musem, the main server of Ente:

![Musem Architecture](https://github.com/ente-io/ente/raw/main/server/scripts/images/museum.png)

The musem server is responsible for managing user authentication, handling file uploads and downloads, and ensuring data consistency across all connected devices. It interacts with various components such as the database, object storage, and encryption modules to provide a seamless and secure storage experience.


### Forking and Modifying Ente
Since Ente.io primarily operates as a subscription-based cloud service, I forked the original repository and stripped away the subscription-related modules to enable self-hosting. Even after removing these modules, I still had to keep placeholder values in my configurations to ensure everything worked seamlessly. The modified version is available here: [https://github.com/jatinkatyal13/ente](https://github.com/jatinkatyal13/ente).

> Faced multiple issues in stripping subscription logic without breaking authentication and storage flows was tricky. I had to carefully trace dependencies and ensure nothing critical was removed. The fork is really dirty is going to be even more so.

### Helm Chart for Ente Deployment
To simplify deployment, I created a Helm chart: [https://github.com/jatinkatyal13/ente-helm](https://github.com/jatinkatyal13/ente-helm). This allows Ente to be deployed in a Kubernetes cluster with minimal configuration.

### Deploying Dependencies with Helmfile
To ensure a fully self-contained setup, I deployed all dependencies—including PostgreSQL, MinIO (for object storage), Longhorn (for persistent storage), and Tailscale (for secure access)—using a Helmfile. The Helmfile structure is as follows:

```yaml
repositories:
  - name: ente-helm
    url: https://jatinkatyal13.github.io/ente-helm
  - name: longhorn
    url: https://charts.longhorn.io
  - name: bitnami
    url: https://charts.bitnami.com/bitnami
  - name: minio
    url: https://charts.min.io/
  - name: onechart
    url: https://chart.onechart.dev

releases:
  - name: ente
    namespace: ente
    chart: ente-helm/ente-helm
    version: 0.1.9
    values:
      - values-ente.yaml

  - name: ente-web
    namespace: ente
    chart: onechart/onechart
    version: 0.73.0
    values:
      - values-ente-web.yaml

  - name: longhorn
    namespace: longhorn-system
    chart: longhorn/longhorn
    version: 1.8.0
    values:
      - values-longhorn.yaml

  - name: postgres
    namespace: database
    chart: bitnami/postgresql
    version: 15.2.2
    values:
      - values-postgres.yaml

  - name: minio
    namespace: minio
    chart: minio/minio
    version: 5.0.15
    values:
      - values-minio.yaml
```

### Reliable Storage with Longhorn
Since I wanted an elastic and portable storage layer, I chose Longhorn for distributed persistent storage across multiple Raspberry Pis. This ensures data redundancy, fault tolerance, and seamless expansion as I add more nodes.

### Accessing Ente Over Tailscale
Finally, I was able to access my self-hosted Ente instance over Tailscale VPN, ensuring secure remote access to my personal cloud.

> Currently, I am still working on building the Ente Android and iOS apps so they can seamlessly connect with my self-hosted setup. This involves modifying the client configs and rebuilding. Which for iOS is not a pleasant experience.


---

## Key Takeaways

1. **Scalability**: Adding new Raspberry Pis increases storage and processing power without complex reconfiguration.
2. **Portability**: With Tailscale and Cloudflare tunnels, I can access my private cloud from anywhere securely.
3. **Abstraction**: The Helm chart and Helmfile make deploying Ente.io and its dependencies easy, even for those unfamiliar with Kubernetes.
4. **Reliability**: Longhorn ensures that even if one Raspberry Pi fails, data remains intact.

This project turned my Raspberry Pi cluster into a truly personal cloud, allowing me to control my data while maintaining cloud-like convenience.

---

## Next Steps

1. Building the client apps to take self hosted URLs as input.
2. Make a simple DIY open source project.

Stay tuned for updates, and feel free to check out the repositories if you want to try this yourself!

- [Ente Fork: Self-Hosted Version](https://github.com/jatinkatyal13/ente)
- [Helm Chart for Ente Deployment](https://github.com/jatinkatyal13/ente-helm)



# Obsidian Deployment on Kubernetes

This documentation provides a guide to deploying the Obsidian on a Kubernetes cluster using Talos and MetalLB, managed via ArgoCD. Each deployment component is explained in detail to ensure a clear understanding of the process and its purpose.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Components](#components)
   - [Deployment](#deployment)
   - [Service](#service)
   - [Persistent Volume and Persistent Volume Claim](#persistent-volume-and-persistent-volume-claim)
4. [Installation Steps](#installation-steps)
5. [Accessing the Application](#accessing-the-application)
6. [Troubleshooting](#troubleshooting)
7. [Conclusion](#conclusion)
8. [Known Issue](#Known-Issue)

## Overview

This project deploys the Obsidian on a Kubernetes cluster, utilizing Talos for Kubernetes management and MetalLB for load balancing. The deployment is managed via ArgoCD, ensuring continuous delivery and version control. This Lab is Running on Proxmox. 

## Prerequisites

Before proceeding, ensure you have the following:

1. **Kubernetes Cluster**: A running Kubernetes cluster, deployed on Talos.
2. **MetalLB**: MetalLB is configured to handle LoadBalancer services.
3. **ArgoCD**: Installed and configured ArgoCD for GitOps deployment.
4. **kubectl & talosctl**: Command-line tool configured to interact with your Kubernetes cluster.
5. **Persistent Storage**: Available storage on the Kubernetes nodes at `/var/mnt/data`.

## Components

### Deployment

The Deployment configuration for the Obsidian service specifies how the application should be deployed and managed. 

- **Replicas**: Specifies the number of pod instances to run. In this setup, only one replica is deployed.
- **Container Image**: The image used for the Obsidian service (`lscr.io/linuxserver/obsidian:latest`).
- **Environment Variables**: Sets necessary environment variables such as user ID, group ID, and time zone.
- **Resource Requests**: Specifies the memory requirements for the container.
- **Volume Mounts**: Mounts the configuration data into the container using a PersistentVolumeClaim.

### Service

The Service configuration exposes the Obsidian deployment to external access via a LoadBalancer.

- **Selector**: Identifies the pods that the service applies to, based on labels.
- **Ports**: Defines the ports for HTTP and WebSocket communication.
- **Type**: Set to `LoadBalancer` to expose the service externally and assign a public IP address via MetalLB.

### Persistent Volume and Persistent Volume Claim

#### Persistent Volume

The PersistentVolume (PV) configuration defines the actual storage resource in the cluster.

- **Capacity**: Specifies the storage capacity (1Gi in this case).
- **Access Modes**: Defines how the volume can be mounted (ReadWriteOnce).
- **PersistentVolumeReclaimPolicy**: Set to `Retain` to retain the data when the PVC is deleted.
- **Host Path**: Specifies the path on the host machine where data is stored (`/var/mnt/data`).

#### Persistent Volume Claim

The PersistentVolumeClaim (PVC) configuration requests the storage resources defined by the PV.

- **Access Modes**: Specifies how the volume can be accessed (ReadWriteOnce).
- **Storage Requests**: Requests a specific amount of storage (1Gi).

## Installation Steps

To deploy the Obsidian service using ArgoCD, follow these steps:

1. **Configure ArgoCD Application**:

   Create an ArgoCD application configuration that points to the repository containing the Kubernetes manifests.

   ```yaml
   project: default
    source:
      repoURL: 'https://your-repo-url.git'
      path: .
      targetRevision: HEAD
    destination:
      server: 'https://kubernetes.default.svc'
      namespace: default
    syncPolicy:
      automated:
        prune: true
        selfHeal: true
      syncOptions:
        - CreateNamespace=true
   ```


2. **Deploy Application via ArgoCD**:

   Once the ArgoCD application is configured and pointing to the correct repository path, it will automatically sync and deploy the resources defined in the manifests.

## Accessing the Application

To access the Obsidian service:

1. **Get the External IP**:

   Retrieve the external IP address assigned to the LoadBalancer service.

   ```bash
   kubectl get svc obsidian
   ```

2. **Access the Application**:

   Open your browser and navigate to `http://<external-ip>:3000` to access the Obsidian service.

## Known Issue: Graph View Crashing

There is a known issue with the Graph View in Obsidian where the screen turns black upon opening the Graph View. This problem is related to the X11 server configuration inside the container.

---
author: "Selwyn Rogers"
title: "Hashicorp Vault in High Availability Mode on RedHat OpenShift 4.10.10"
date: "2023-03-06"
description: "This guide provides a comprehensive, step-by-step tutorial for setting up Hashicorp Vault on Openshift in High Availability mode, utilizing the integrated Raft storage system."
tags: ["openshift", "redhat","vault"]
ShowToc: false
ShowBreadCrumbs: false
---

Hashicorp Vault is a widely-used open-source tool for securely storing, managing, and accessing secrets, such as passwords, API keys, and certificates. It provides a centralized platform for managing secrets across different environments and integrates with various authentication and authorization systems. Vault offers features such as dynamic secrets, secret leasing, and revocation, making it a popular choice for managing secrets in modern application architectures.

Configuring Hashicorp Vault on Openshift in High Availability mode can be a challenging task, as the official documentation may not provide all the necessary information.

### Add Helm repository
To proceed, add the Hashicorp Helm repository as an official source.
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm search repo hashicorp/vault
```
### Helmchart Configuration
Create a local project directory.
```bash
mkdir /srv/vault
cd /srv/vault
```
First download the values.yaml file example from the official Hashicorp Vault GitHub repository.
This file will serve as a template for any further customizations.
```bash
wget https://raw.githubusercontent.com/hashicorp/vault-helm/main/values.yaml
```
Next, edit the `values.yaml` with the editor of your choice. 
```bash
vim values.yaml
```
#### OpenShift support
There are several OpenShift-specific values that need to be set.

Set the `openshift` value to true:
```yaml
  openshift: true
``` 

#### Vault Agent Injector (optional)
This guide does not cover the Vault Agent Injector, as it falls outside the scope of this tutorial.
If you plan to utilize the Vault Agent Injector, you may skip this step. Otherwise set the `enabled` value to `false`.
```yaml
injector:
  # True if you want to enable vault agent injection.
  # @default: global.enabled
  enabled: "false"
```
#### Vault Server Settings
##### Enterprise License Key (optional)
If you possess a Vault Enterprise license, you must add the license key as an Openshift secret. This can be accomplished by navigating to the `enterpriseLicense` block, located within the `server` block, within the `values.yaml` file.
Please note that this guide employs the community edition of Vault.

```yaml
server:
....
  enterpriseLicense:
    # The name of the Kubernetes secret that holds the enterprise license. The
    # secret must be in the same namespace that Vault is installed into.
    secretName: ""
    # The key within the Kubernetes secret that holds the enterprise license.
    secretKey: "license"
...
```

##### Vault Image Version
You may consider modifying the image tag to a particular version of Vault.
To achieve this, locate the `image` block within the `server` block within in the `values.yaml` file and modify the value of `tag` to reflect the desired version.
You can find the latest tags for Hashicorp Vault on the official Docker Hub registry at: https://hub.docker.com/r/hashicorp/vault/tags.

```yaml
server:
...
  image:
    repository: "hashicorp/vault"
    tag: "1.12.1"
...
```

##### Log Level
For production environments set the log level to `info`. For testing `debug` can come handy.
```yaml
...
  # Configure the logging verbosity for the Vault server.
  # Supported log levels include: trace, debug, info, warn, error
  logLevel: "info"
...
```

##### Server Ressources
Locate the `resources` key within the `server`  block and adjust the values based on your requirements.
For this example, the default values have been utilized.

```yaml
server:
...
  resources:
    requests:
      memory: 256Mi
      cpu: 250m
    limits:
      memory: 256Mi
      cpu: 250m
...
```
##### OpenShift Route
OpenShift Routes are a type of Kubernetes Ingress resource that exposes services running inside OpenShift to the external network. They enable external clients to communicate with a service, without requiring knowledge of the internal IP addresses or network topology. OpenShift Routes can perform SSL termination, load balancing, and traffic routing based on various criteria, such as path or header values.

Kubernetes also provides a similar resource called Ingress to expose services to the external network. The main difference between OpenShift Routes and Kubernetes Ingress is that OpenShift Routes are OpenShift-specific and provide additional features, such as built-in TLS termination and support for advanced traffic routing configurations. However, both resources serve a similar purpose of exposing Kubernetes services to the outside world.

This particular route will serve as the URL for accessing the Vault web user interface.

If you do not require a web interface, feel free to skip this step.

Otherwise, navigate to the `route` section within the `values.yaml`file and set the `enabled` value to true, configure the hostname to your preferred domain, and set `tls: termination: passthrough` to `edge`. 


```yaml
  # OpenShift only - create a route to expose the service
  # By default the created route will be of type passthrough
  route:
    enabled: true

    # When HA mode is enabled and K8s service registration is being used,
    # configure the route to point to the Vault active service.
    activeService: true

    labels: {}
    annotations: {}
    host: vault.apps.cluster-example.local
    # tls will be passed directly to the route's TLS config, which
    # can be used to configure other termination methods that terminate
    # TLS at the router
    tls:
      termination: edge
```
###### TLS Certificate
For this example, an internal cluster subdomain has been utilized as the hostname. However, if you utilize a different domain, you will need to add the TLS certificate and key for the route. This can be easily accomplished via the OpenShift Web Console, by navigating to "Networking" and "Routes" after the installation is completed. Simply add the certificate and key to the corresponding fields within the route's YAML manifest.


##### Data and Audit Storage
To ensure the persistence of data and audit logs, you must configure the appropriate data and audit storage mechanisms. To achieve this, configure the `storageClass` in accordance with the examples below.

###### Storage Class
The storage classes available to you may differ based on the particular storage backend you are utilizing. Therefore, the available storage classes may vary based on your individual circumstances.

```yaml
  dataStorage:
    enabled: true
    # Size of the PVC created
    size: 10Gi
    # Location where the PVC will be mounted.
    mountPath: "/vault/data"
    # Name of the storage class to use.  If null it will use the
    # configured default Storage Class.
    storageClass: local-storage
    # Access Mode of the storage device being used for the PVC
    accessMode: ReadWriteOnce
    # Annotations to apply to the PVC
    annotations: {}
```

```yaml
  auditStorage:
    enabled: false
    # Size of the PVC created
    size: 10Gi
    # Location where the PVC will be mounted.
    mountPath: "/vault/audit"
    # Name of the storage class to use.  If null it will use the
    # configured default Storage Class.
    storageClass: local-storage
    # Access Mode of the storage device being used for the PVC
    accessMode: ReadWriteOnce
    # Annotations to apply to the PVC
    annotations: {}
```

In the example the `storageClass` is set to `local-storage`.
The `local-storage` storage class in OpenShift allows you to utilize the local storage of each worker node for your applications. This can be useful for scenarios where you need to store data that does not require high availability or data replication. The local-storage class provisions storage from the local disk of each worker node, rather than relying on a shared storage system. However, this approach also comes with certain limitations, such as limited capacity and potential data loss in the event of node failure. Therefore, it is important to carefully evaluate the use case and requirements before utilizing the local-storage class.

Raft integrated storage can provide replication across different nodes using local storage. While using the local-storage storage class with Raft replication may be suitable for some small-scale or non-critical production environments, it may not be the most robust solution for larger or mission-critical systems.

Some alternative storage solutions that are designed to provide higher durability and availability include network-attached storage (NAS) systems, distributed file systems such as Ceph or GlusterFS, or cloud-based storage solutions such as Amazon S3 or Google Cloud Storage.

In my experience, I have not utilized the local-storage storage class, as I am currently utilizing the filesystem storage class on RedHat ODF (OpenShift Data Foundation), which is built on the Ceph distributed file system. This solution provides superior durability, availability, and scalability compared to local storage, and includes several additional features and capabilities, such as data replication, snapshotting, and encryption, that are crucial for production environments.

##### Developer Mode
This setting will let you run Vault in Developer Mode. This is useful for debugging but should never be used in a production environment. Make sure it is set to `false`

```yaml
  dev:
    enabled: false
```

###### High Availabilty Mode
By default, HA mode is disabled in Vault. However, for production environments, it is recommended to run Vault in HA mode for improved availability and redundancy. To enable HA mode, set the enabled value to true and, depending on the number of OpenShift worker nodes, configure the replicas value to a minimum of 3.
```yaml
  ha:
    enabled: true
    replicas: 3
```
###### Raft Integrated Storage
Raft storage is an integrated storage mechanism offered by Hashicorp Vault that provides a built-in high availability solution for storing Vault data. Raft uses a distributed consensus algorithm to replicate data across multiple Vault servers, ensuring that data remains available even in the event of server failures or network outages. It eliminates the need for an external storage system, such as Hashicorp Consul or a database, which simplifies the deployment and configuration of Vault clusters. Raft storage is recommended for production environments that require high availability and data redundancy.

By default, the Helm chart is configured to use Hashicorp Consul as the storage backend. However, in this guide, we will be using Raft integrated storage instead, as it offers a more straightforward setup. To accomplish this, set the enabled value to true, which will overwrite Consul as the default storage mechanism.


```yaml
    # Enables Vault's integrated Raft storage.  Unlike the typical HA modes where
    # Vault's persistence is external (such as Consul), enabling Raft mode will create
    # persistent volumes for Vault to store data according to the configuration under server.dataStorage.
    # The Vault cluster will coordinate leader elections and failovers internally.
    raft:

      # Enables Raft integrated storage
      enabled: true
      # Set the Node Raft ID to the name of the pod
      setNodeId: false
```

##### Vault UI

By default, the Web UI is disabled in the Helm Chart. To enable it, navigate to the values.yaml file and set enabled to true within the ui block.

```yaml
# Vault UI
ui:
  # True if you want to create a Service entry for the Vault UI.
  #
  # serviceType can be used to control the type of service created. For
  # example, setting this to "LoadBalancer" will create an external load
  # balancer (for supported K8S installations) to access the UI.
  enabled: true
  publishNotReadyAddresses: true
  # The service should only contain selectors for active Vault pod
  activeVaultPodOnly: false
  serviceType: "ClusterIP"
  serviceNodePort: null
  externalPort: 8200
  targetPort: 8200
```

#### Keep in mind
It is important to note that there are many additional settings available within the values.yaml file that may be relevant to your particular use case. The settings presented in this guide are those that I have personally utilized in production and have found to be effective within my work environment. However, it is always recommended to review the full set of available options and configure them based on your specific requirements and constraints.

## Installation on OpenShift

### OpenShift Namespace
To begin, create a new namespace on OpenShift. For this example, the namespace has been named "vault".
```bash
oc create namespace vault
oc project vault
```

### Deploy using Helm
```bash
cd /srv/vault

helm install vault hashicorp/vault \
  -f values.yaml \
  --namespace vault
```

### Initialize Cluster
After the vault containers are created, they do not immediately transition to the ready state. To prepare them for use, the cluster must first be initiated on the first pod vault-0. Then, the vault pods must be unsealed using the keys provided after initializing the cluster. Finally, the pods must be joined to the cluster for full functionality.

To initialize the cluster run following command:
```bash
oc exec --stdin=true --tty=true vault-0 -- vault operator init
```
This will initialize the cluster and generate five unseal keys and a root token, which will be displayed in the console output. Please make sure to take note of them.

Example Output:
```bash
Unseal Key 1: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Unseal Key 2: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Unseal Key 3: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Unseal Key 4: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Unseal Key 5: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

Initial Root Token: hvs.XXXXXXXXXXXXXXXXXXXXXXXX
```

#### Unseal nodes and join to cluster
1. To begin, you will need to unseal the first node, vault-0, using each of the five provided unseal keys. To do this, run the following command five times in a row, using a different unseal key each time. Please ensure that each unseal key is used only once. You will be prompted for the key:

```bash
oc exec -ti vault-0 -- vault operator unseal
oc exec -ti vault-0 -- vault operator unseal
oc exec -ti vault-0 -- vault operator unseal
oc exec -ti vault-0 -- vault operator unseal
oc exec -ti vault-0 -- vault operator unseal
````
2. To proceed, join the second node, vault-1, to the cluster by running the following command
```bash
oc exec -ti vault-1 -- vault operator raft join http://vault-0.vault-internal:8200
```
3. Afterward, repeat the unsealing process with vault-1 in the same way as before with vault-0. Make sure to use the same Unseal Keys in the same order as you did with vault-0.
```bash
oc exec -ti vault-1 -- vault operator unseal
oc exec -ti vault-1 -- vault operator unseal
oc exec -ti vault-1 -- vault operator unseal
oc exec -ti vault-1 -- vault operator unseal
oc exec -ti vault-1 -- vault operator unseal
```
4. Now, joint the third node, vault-2, to the cluster:
```bash
oc exec -ti vault-2 -- vault operator raft join http://vault-0.vault-internal:8200
```
5. Finally, unseal vault-2 the exact same way as you did with vault-1 
```bash
oc exec -ti vault-2 -- vault operator unseal
oc exec -ti vault-2 -- vault operator unseal
oc exec -ti vault-2 -- vault operator unseal
oc exec -ti vault-2 -- vault operator unseal
oc exec -ti vault-2 -- vault operator unseal
````
6. Check the ready state of the pods: `oc get pods`
7. Navigate to the route defined in the values.yaml file in your browser, and log in to the Vault UI using the root token.
8. Congratulations, you now have a Vault cluster set up in HA mode.

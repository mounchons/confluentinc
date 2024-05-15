To create a Kafka cluster using the Bitnami Kafka Helm chart with 3 brokers, 3 ZooKeepers, TLS mode, and high availability configuration, you can follow these steps:

### 1. Install Helm (if not already installed)
If you don’t have Helm installed, you can install it by following the instructions [here](https://helm.sh/docs/intro/install/).

### 2. Add the Bitnami repository
Add the Bitnami charts repository to your Helm configuration:
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### 3. Create a `values.yaml` file
Create a `values.yaml` file to specify your custom configuration for the Kafka cluster.

### Example `values.yaml`
Here is an example `values.yaml` that configures 3 Kafka brokers and 3 ZooKeepers, with TLS enabled:

```yaml
global:
  storageClass: "nfs"

zookeeper:
  enabled: true
  replicas: 3
  auth:
    clientUser: "zk-client"
    clientPassword: "zk-client-password"
    serverUsers:
      - "zk-server"
    serverPasswords:
      - "zk-server-password"
  persistence:
    enabled: true
    storageClass: "nfs"
    accessModes:
      - ReadWriteMany
    size: 100Mi
  resources:
    limits:
      memory: 512Mi
      cpu: 500m
    requests:
      memory: 256Mi
      cpu: 250m

kafka:
  replicas: 3
  auth:
    clientProtocol: tls
    interBrokerProtocol: tls
    tls:
      enabled: true
      existingSecret: kafka-tls-secret # Ensure you have a Kubernetes secret with your TLS certificates
  persistence:
    enabled: true
    storageClass: "nfs"
    accessModes:
      - ReadWriteMany
    size: 100Mi
  externalAccess:
    enabled: true
    service:
      type: NodePort
    autoDiscovery:
      enabled: true
  resources:
    limits:
      memory: 1024Mi
      cpu: 1000m
    requests:
      memory: 512Mi
      cpu: 500m
  livenessProbe:
    enabled: true
    initialDelaySeconds: 60
    periodSeconds: 10
  readinessProbe:
    enabled: true
    initialDelaySeconds: 60
    periodSeconds: 10

metrics:
  kafka:
    enabled: true
    serviceMonitor:
      enabled: true
```

### 4. Create a Kubernetes Secret for TLS Certificates
You need to create a Kubernetes secret that contains your TLS certificates. Here’s an example of how to create such a secret:

```sh
kubectl create secret generic kafka-tls-secret \
  --from-file=ca.crt \
  --from-file=broker.crt \
  --from-file=broker.key
```

### 5. Install the Kafka Helm Chart
Use the `values.yaml` file to install the Kafka chart with the specified configuration:

```sh
helm upgrade --install kafka bitnami/kafka -f values.yaml --namespace dev --create-namespace
```

### Summary
1. **Install Helm**: Ensure Helm is installed and the Bitnami repository is added.
2. **Create `values.yaml`**: Configure the Kafka cluster with 3 brokers, 3 ZooKeepers, TLS mode, and HA settings.
3. **Create TLS Secret**: Create a Kubernetes secret with your TLS certificates.
4. **Install Kafka**: Use the Helm chart to deploy the Kafka cluster.

By following these steps, you will set up a Kafka cluster with the desired configuration. Adjust the paths to your TLS certificates and other parameters as needed.
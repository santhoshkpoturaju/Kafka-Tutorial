# Single Instance - Docker/Podman Based Installation

This guide demonstrates how to set up a single-node Apache Kafka instance using containerization. This approach is ideal for local development, testing, and proof-of-concept environments.

**Baseline Reference:** https://kafka.apache.org/quickstart

## Important Note on Podman vs Docker

This tutorial uses **Podman** for container management. However, the commands are nearly identical for Docker. Podman is a daemonless, open-source container engine that works similarly to Docker but with some architectural differences.

Throughout this guide, I demonstrate both approaches:

### For Docker-based Installation
If you prefer Docker, simply replace `podman` with `docker` in all commands:

```bash
docker pull apache/kafka-native:4.1.1
```

### For Podman-based Installation
For Podman users, use the following command:

```bash
podman pull apache/kafka-native:4.1.1
```

**Note:** The rest of this documentation follows Podman-based instructions. Substitute `podman` with `docker` and `podman-compose` with `docker-compose` as needed for your container runtime of choice. 


## Apache Kafka Installation

This section guides you through setting up a single Kafka broker with KRaft (Kafka Raft Metadata) mode, which is the modern approach replacing ZooKeeper for cluster coordination.

### Step 1: Create a Network for Container Communication
```bash
podman network create kafkanet
```
This command creates an isolated network named `kafkanet` that allows containers to communicate with each other using hostnames. This is essential for Kafka and its UI to discover and communicate with each other.

### Step 2: Pull the Kafka Image
```bash
podman pull apache/kafka:4.1.1
```
Downloads the official Apache Kafka image (version 4.1.1) from the container registry. This base image contains all necessary Kafka binaries and dependencies.

### Step 3: Run the Kafka Broker Container
```bash
podman run -d --name kafka \
  --network kafkanet \
  -p 9092:9092 \
  -e KAFKA_NODE_ID=1 \
  -e KAFKA_PROCESS_ROLES=broker,controller \
  -e KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:29093 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092 \
  -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@kafka:29093 \
  -e CLUSTER_ID=5L6g3nShT-eMCtK--X86sw \
  apache/kafka:4.1.1
```

**Configuration Breakdown:**
- `-d`: Runs the container in detached mode (background)
- `--name kafka`: Assigns the name "kafka" to the container for easy reference
- `--network kafkanet`: Connects the container to the previously created network
- `-p 9092:9092`: Maps port 9092 from the container to the host, allowing producers/consumers to connect
- `KAFKA_NODE_ID=1`: Unique identifier for this broker in the cluster
- `KAFKA_PROCESS_ROLES=broker,controller`: This node acts as both a broker and KRaft controller
- `KAFKA_LISTENERS`: Internal communication endpoints (PLAINTEXT for clients, CONTROLLER for KRaft)
- `KAFKA_ADVERTISED_LISTENERS`: The address advertised to external clients (using the service name "kafka")
- `KAFKA_CONTROLLER_LISTENER_NAMES`: Designates which listener is used for KRaft communication
- `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP`: Maps listener names to security protocols
- `KAFKA_CONTROLLER_QUORUM_VOTERS`: Specifies the controller node for KRaft quorum
- `CLUSTER_ID`: Unique identifier for the Kafka cluster

## Kafka UI Installation

Kafka UI is a web-based graphical interface that provides visibility into your Kafka cluster. It allows you to browse topics, partitions, consumer groups, and view messages without using command-line tools.

```bash
podman run -d --name kafka-ui \
  --network kafkanet \
  -p 8080:8080 \
  -e DYNAMIC_CONFIG_ENABLED=true \
  provectuslabs/kafka-ui
```

**Configuration Breakdown:**
- `-d`: Runs the container in detached mode
- `--name kafka-ui`: Names the container for easy reference
- `--network kafkanet`: Connects to the same network as Kafka, allowing automatic service discovery
- `-p 8080:8080`: Maps port 8080 (accessible at `http://localhost:8080`)
- `DYNAMIC_CONFIG_ENABLED=true`: Allows dynamic cluster configuration through the UI
- `provectuslabs/kafka-ui`: Official Kafka UI image maintained by Provectus

**Accessing Kafka UI:** Once running, open your browser and navigate to `http://localhost:8080` to access the dashboard.

## Using Docker Compose for Simplified Setup

While the individual container commands above work well for understanding the components, managing multiple containers manually can become tedious and error-prone. This is where Docker Compose and Podman Compose come in handy.

**Why Use Compose Files?**
- Declarative infrastructure as code
- Single command to start/stop entire stack
- Automatic network creation and linking
- Simplified container management and debugging
- Easy to version control and share with team members

The complete configuration for both Kafka and Kafka UI has been combined into a single `docker-compose.yaml`. Those compose files located in the `/installation` directory. these compose files encapsulate all the environment variables, network settings, and port mappings needed for a fully functional Kafka development environment.

### Starting the Stack

To spin up your entire Kafka and UI environment with a single command:

```bash
podman compose --file docker-compose.yaml up -d # for podman
docker-compose up -d # for docker
```

**What this command does:**
- `up`: Creates and starts all services defined in the compose file
- `-d`: Runs in detached mode (services run in the background)

This single command will:
1. Create the shared network
2. Pull the required images
3. Start the Kafka broker container
4. Start the Kafka UI container
5. Configure all environment variables and port mappings

### Stopping the Stack

To stop all services:

```bash
podman compose --file docker-compose.yaml down # for podman
docker-compose down # for docker
```

**Review the Docker Compose Configuration:**

Examine the `docker-compose.yaml` files in the `/installation` directory to understand the complete setup and customize it for your needs.


# Single Instance - Kubernetes Based

## Using Manifest File

Run the 1.manifest.yaml file to spin up the kafka cluster which is going to setup single node kafka cluster and also Kafka UI. To apply this manifest file, run the following command

```bash
kubectl apply -f 1.manifest.yaml
kubectl get pods -n kafka-lab
kubectl port-forward -n kafka-lab svc/kafka 9092:9092 19092:19092 &
kubectl port-forward -n kafka-lab svc/kafka-ui 8080:8080 &
```


# Helm-Based Deployment

Helm is a package manager for Kubernetes that simplifies complex deployments through templated charts. This section covers deploying Apache Kafka to a Kubernetes cluster using Helm and the Bitnami Kafka chart.

## Prerequisites

Before proceeding with Helm deployment, ensure you have:
- A running Kubernetes cluster (local or cloud-based)
- `kubectl` configured and authenticated to your cluster
- Helm 3.x installed on your local machine
- Sufficient cluster resources (CPU, memory, storage)

## Understanding Helm Charts

A Helm chart is a collection of YAML templates that define Kubernetes resources. The Bitnami Kafka chart provides:
- Kafka brokers with KRaft mode
- ConfigMaps for configuration management
- StatefulSets for stateful workload management
- Services for network exposure
- PersistentVolumeClaims for data persistence
- RBAC (Role-Based Access Control) definitions

## Adding the Bitnami Repository

The Bitnami repository contains production-ready Helm charts maintained by VMware. Before deploying Kafka, add this repository to your Helm installation:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

This command registers the Bitnami charts repository, making all their charts available for installation.

## Updating Helm Repository Cache

After adding the repository, update your local Helm cache to fetch the latest chart versions:

```bash
helm repo update
```

This ensures you have access to the most recent versions of available charts and security patches.

## Deploying Kafka Using Helm

Deploy the Kafka cluster using the Bitnami chart with a custom values configuration file:

```bash
helm install kafka oci://registry-1.docker.io/bitnamicharts/kafka -n kafka-lab --create-namespace -f installation/kubernetes-based/2.values.kafka.yaml 
```

**Command Breakdown:**
- `helm install`: Installs a new release
- `kafka`: Name of the Helm release (used for managing, upgrading, and uninstalling)
- `oci://registry-1.docker.io/bitnamicharts/kafka`: OCI registry reference to the Bitnami Kafka chart
- `-n kafka-lab`: Specifies the Kubernetes namespace where Kafka will be deployed (created if it doesn't exist)
- `--create-namespace`: Creates the namespace if it doesn't already exist
- `-f installation/kubernetes-based/2.values.kafka.yaml`: Path to custom values file that overrides default chart configuration

## Custom Values File

The `2.values.kafka.yaml` file contains customizations specific to your environment, such as:
- Number of Kafka replicas
- Resource requests and limits (CPU, memory)
- Storage configuration
- Authentication and authorization settings
- Listener and advertised listener configurations

Review and modify this file according to your deployment requirements before running the helm install command. But for simiplicity, i kept everything default, except image path which is mandatory as per bitmani strcucture change.

## Monitoring Deployment Progress

After running the helm install command, monitor the deployment status:

```bash
kubectl get pods -n kafka-lab
```

Wait until all Kafka pods show `Running` status, which may take several minutes depending on your cluster resources.

# Sample Output as given below
```bash
varma@macoss-MacBook-Pro Kafka-Tutorial % helm install kafka oci://registry-1.docker.io/bitnamicharts/kafka -n kafka-lab --create-namespace -f installation/kubernetes-based/2.values.kafka.yaml 
Pulled: registry-1.docker.io/bitnamicharts/kafka:32.4.3
Digest: sha256:12b98a1b358a6bc10c498817c801bd49e4a9d4c965af8acbe5a70764ec836997
I1208 19:29:35.231853   25468 warnings.go:110] "Warning: spec.SessionAffinity is ignored for headless services"
NAME: kafka
LAST DEPLOYED: Mon Dec  8 19:29:34 2025
NAMESPACE: kafka-lab
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
CHART NAME: kafka
CHART VERSION: 32.4.3
APP VERSION: 4.0.0

⚠ WARNING: Since August 28th, 2025, only a limited subset of images/charts are available for free.
    Subscribe to Bitnami Secure Images to receive continued support and security updates.
    More info at https://bitnami.com and https://github.com/bitnami/containers/issues/83267

** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    kafka.kafka-lab.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    kafka-controller-0.kafka-controller-headless.kafka-lab.svc.cluster.local:9092
    kafka-controller-1.kafka-controller-headless.kafka-lab.svc.cluster.local:9092
    kafka-controller-2.kafka-controller-headless.kafka-lab.svc.cluster.local:9092

The CLIENT listener for Kafka client connections from within your cluster have been configured with the following security settings:
    - SASL authentication

To connect a client to your Kafka, you need to create the 'client.properties' configuration files with the content below:

security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="user1" \
    password="$(kubectl get secret kafka-user-passwords --namespace kafka-lab -o jsonpath='{.data.client-passwords}' | base64 -d | cut -d , -f 1)";

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run kafka-client --restart='Never' --image docker.io/bitnamilegacy/kafka:4.0.0-debian-12-r10 --namespace kafka-lab --command -- sleep infinity
    kubectl cp --namespace kafka-lab /path/to/client.properties kafka-client:/tmp/client.properties
    kubectl exec --tty -i kafka-client --namespace kafka-lab -- bash

    PRODUCER:
        kafka-console-producer.sh \
            --producer.config /tmp/client.properties \
            --bootstrap-server kafka.kafka-lab.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --consumer.config /tmp/client.properties \
            --bootstrap-server kafka.kafka-lab.svc.cluster.local:9092 \
            --topic test \
            --from-beginning

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - controller.resources
  - defaultInitContainers.prepareConfig.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

⚠ SECURITY WARNING: Original containers have been substituted. This Helm chart was designed, tested, and validated on multiple platforms using a specific set of Bitnami and Tanzu Application Catalog containers. Substituting other containers is likely to cause degraded security and performance, broken chart features, and missing environment variables.

Substituted images detected:
  - docker.io/bitnamilegacy/kafka:4.0.0-debian-12-r10

⚠ WARNING: Original containers have been substituted for unrecognized ones. Deploying this chart with non-standard containers is likely to cause degraded security and performance, broken chart features, and missing environment variables.

Unrecognized images:
  - docker.io/bitnamilegacy/kafka:4.0.0-debian-12-r10
varma@macoss-MacBook-Pro Kafka-Tutorial % 
```

# Kafka UI Installation

While Helm charts for Kafka UI are available, this guide uses a Kubernetes manifest file for simplicity and consistency with the earlier sections. However, you can optionally deploy Kafka UI via Helm by searching for community or Bitnami-maintained Helm charts.

## Deploying Kafka UI via Manifest

Apply the Kafka UI manifest file to deploy the web interface into your Kubernetes cluster:

```bash
kubectl apply -f installation/kubernetes-based/3.kafka-ui-manifest.yaml
```

This command creates all necessary Kubernetes resources (Deployment, Service, ConfigMap) defined in the manifest file.

## Accessing Kafka UI

After the Kafka UI pod is running, create a port-forward to access the web interface locally:

```bash
kubectl port-forward -n kafka-ui svc/kafka-ui 8080:8080
```

**What this command does:**
- `port-forward`: Establishes a tunnel from your local machine to the Kafka UI service
- `-n kafka-ui`: Specifies the namespace where Kafka UI is running
- `svc/kafka-ui`: Forwards to the kafka-ui service
- `8080:8080`: Maps local port 8080 to service port 8080

Once the port-forward is established, open your browser and navigate to `http://localhost:8080` to access the Kafka UI dashboard.

## Verifying Kafka UI Deployment

Check that the Kafka UI pod is running:

```bash
kubectl get pods -n kafka-ui -l app=kafka-ui
```

View the Kafka UI logs for debugging:

```bash
kubectl logs -n kafka-ui -l app=kafka-ui
```



## Connecting to Kafka Cluster

The Helm deployment configured Kafka with SASL (Simple Authentication and Security Layer) authentication, requiring credentials to connect. This section provides the necessary steps to authenticate and establish connections.

### Retrieving Authentication Credentials

Extract the client password from the Kubernetes secret created during Helm installation:

```bash
$(kubectl get secret kafka-user-passwords --namespace kafka-lab -o jsonpath='{.data.client-passwords}' | base64 -d | cut -d , -f 1)
```

**Command Breakdown:**
- `kubectl get secret`: Retrieves the secret containing user passwords
- `kafka-user-passwords`: The name of the secret created by the Helm chart
- `--namespace kafka-lab`: Specifies the namespace where the secret is stored
- `-o jsonpath='{.data.client-passwords}'`: Extracts the client-passwords field
- `| base64 -d`: Decodes the base64-encoded password
- `| cut -d , -f 1`: Extracts the first password (if multiple exist)

### Creating a Kafka Client Pod

To interact with the Kafka cluster from within Kubernetes, create a dedicated client pod:

```bash
kubectl run kafka-client --restart='Never' --image docker.io/bitnamilegacy/kafka:4.0.0-debian-12-r10 --namespace kafka-lab --command -- sleep infinity
```

This pod remains running indefinitely, allowing you to execute Kafka commands inside the cluster network.

### Copying Client Configuration

Transfer the client configuration file (containing SASL credentials) to the client pod:

```bash
kubectl cp --namespace kafka-lab /path/to/client.properties kafka-client:/tmp/client.properties
```

Replace `/path/to/client.properties` with the actual path to your client configuration file on your local machine.

### Accessing the Client Pod

Execute an interactive shell within the client pod:

```bash
kubectl exec --tty -i kafka-client --namespace kafka-lab -- bash
```

This command opens a bash shell inside the pod, allowing you to run Kafka commands.

### Producing Messages

Once inside the client pod, use the Kafka producer to send messages:

```bash
kafka-console-producer.sh \
    --producer.config /tmp/client.properties \
    --bootstrap-server kafka.kafka-lab.svc.cluster.local:9092 \
    --topic test
```

**Configuration Details:**
- `--producer.config`: Path to the client properties file with SASL credentials
- `--bootstrap-server`: Kubernetes internal DNS name of the Kafka cluster
- `--topic test`: Target topic for message production

### Consuming Messages

Use the Kafka consumer to read messages from a topic:

```bash
kafka-console-consumer.sh \
    --consumer.config /tmp/client.properties \
    --bootstrap-server kafka.kafka-lab.svc.cluster.local:9092 \
    --topic test \
    --from-beginning
```

**Configuration Details:**
- `--consumer.config`: Path to the client properties file with SASL credentials
- `--bootstrap-server`: Kubernetes internal DNS name of the Kafka cluster
- `--topic test`: Source topic to consume messages from
- `--from-beginning`: Reads all messages from the topic beginning (useful for testing)

### Client Configuration File Template

Create a `client.properties` file with the following content, replacing the password placeholder:

```
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="user1" \
    password="<YOUR_PASSWORD>";
```

Follow the video tutorials included in this repository for step-by-step instructions on connecting to the Kafka cluster via the UI and performing producer/consumer operations.
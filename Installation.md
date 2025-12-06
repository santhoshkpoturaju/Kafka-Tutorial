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

The complete configuration for both Kafka and Kafka UI has been combined into a single `docker-compose.yaml` file located in the `/installation` directory. This compose file encapsulates all the environment variables, network settings, and port mappings needed for a fully functional Kafka development environment.

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

Examine the `docker-compose.yaml` file in the `/installation` directory to understand the complete setup and customize it for your needs.
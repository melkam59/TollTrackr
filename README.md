# Toll Calculator Microservices

A Go-based, event-driven microservices architecture for tracking vehicle movements and calculating toll invoices based on distance traveled. It leverages Kafka for high-throughput event streaming, gRPC for fast inter-service communication, and Prometheus for monitoring.

## 🏗️ Architecture & Services

The application is divided into several decentralized microservices:

1. **OBU (On-Board Unit)**: Simulates a vehicle's transponder. Generates and sends continuous GPS/telemetry data (coordinates) to the receiver.
2. **Data Receiver**: The ingestion layer. Receives coordinates from OBUs and produces/publishes this data to an Apache Kafka topic for downstream services.
3. **Distance Calculator**: A Kafka consumer that reads the stream of location data, calculates the total distance traveled by each specific vehicle, and forwards this data to the Aggregator.
4. **Aggregator**: The central data store and billing engine. Receives calculated distances via gRPC, aggregates the total distance per vehicle, calculates the final toll invoice, and exposes an HTTP/gRPC API to fetch invoices.
5. **Gateway**: An API Gateway acting as the single entry point for clients (e.g., fetching a vehicle's toll invoice) routing requests to the Aggregator.
6. **Go-Kit Example**: Demonstrates the use of the `go-kit` library as an alternative architecture strategy.

## 🚀 Tech Stack

* **Language:** Golang (Go)
* **Message Broker:** Apache Kafka (with Zookeeper)
* **Internal Communication:** gRPC & Protocol Buffers (protobuf)
* **Metrics & Monitoring:** Prometheus
* **Infrastructure:** Docker & Docker Compose

---

## 🛠️ Getting Started

### 1. Prerequisites
- Docker & Docker Compose
- Go 1.18+
- `make`

### 2. Run Kafka & Zookeeper
Start the Confluent Kafka broker and Zookeeper instances:
```bash
docker-compose up -d
```
*(Alternatively, you can run a standalone Bitnami Kafka container)*:
```bash
docker run --name kafka -p 9092:9092 -e ALLOW_PLAINTEXT_LISTENER=yes -e KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true bitnami/kafka:latest 
```

### 3. Running the Microservices
The project includes a `Makefile` to quickly build and run individual services. Open separate terminal tabs for each:

* **Aggregator Service:** `make agg`
* **Distance Calculator:** `make calculator`
* **Data Receiver:** `make receiver`
* **Gateway API:** `make gate`
* **OBU (Data Generator):** `make obu`

---

## ⚙️ Development Setup

### Protobuf Compiler Installation
If you intend to modify the `.proto` files in `types/`, you must install the `protoc` compiler.

**Linux (or WSL2):**
```bash
sudo apt install -y protobuf-compiler
```

**macOS:**
```bash
brew install protobuf
```

### Go Plugins for gRPC
Install the protobuf and gRPC code generator plugins for Go:
```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2

# Ensure your GOPATH bin is in your system PATH
export PATH="${PATH}:${HOME}/go/bin"

# Fetch project dependencies
go get google.golang.org/protobuf
go get google.golang.org/grpc/
```

To re-generate the protobuf code after making changes to `types/ptypes.proto`:
```bash
make proto
```

---

## 📈 Monitoring with Prometheus

**1. Run via Docker (Recommended)**
Mount the local `.config/prometheus.yml` into a prometheus container:
```bash
docker run -p 9090:9090 -v ./.config/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

**2. Running Natively**
```bash
git clone https://github.com/prometheus/prometheus.git
cd prometheus
make build
./prometheus --config.file=../.config/prometheus.yml # adjust to your path
```

*(Prometheus golang client is already included in `go.mod`)*

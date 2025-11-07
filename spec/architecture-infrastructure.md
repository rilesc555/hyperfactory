# Architecture & Infrastructure

## Decided

- **Cloud Platform**: Hyperfactory instances will run in the cloud on OpenMetal infrastructure
- **Edge Technology**: Will fork EdgeX Foundry to run on factory edge devices/boxes
- **Container Orchestration**: Docker Compose for simplicity, migrate to Kubernetes when scaling demands require it
- **Data Flow**: Edge devices stream data to hyperfactory instances running in the cloud
- **Infrastructure Context**: There is existing infrastructure code in the /context folder from another project that controls a Kubernetes cluster, which may or may not be used for this project
- **Architecture Philosophy**: Keep things simple for small team/product, but maintain ability to scale when necessary
- **Scaling Priority**: Factory data flow is relatively steady, so immediate elasticity is not the highest priority

## Undecided

### 1. Container Orchestration: Kubernetes vs. Simpler Approaches

**Kubernetes Pros:**
- Mature ecosystem with extensive tooling
- Excellent for scaling and multi-tenancy
- Strong service discovery and networking
- Existing infra code in /context folder
- Industry standard for cloud-native apps

**Kubernetes Cons:**
- Complex for small team/simple product
- Higher operational overhead
- Steeper learning curve
- Over-engineering for steady factory workloads

**Docker Compose Pros:**
- Simple to understand and deploy
- Perfect for small team/product
- Fast development iteration
- Minimal operational overhead
- Good for steady, predictable workloads

**Docker Compose Cons:**
- Limited scaling capabilities
- No built-in service discovery
- Single-host limitation
- Manual multi-tenancy implementation

**Nomad Pros:**
- Simpler than K8s but more powerful than Compose
- Good scaling capabilities
- Multi-host support
- Lower operational overhead than K8s

**Nomad Cons:**
- Smaller ecosystem
- Less tooling available
- Team learning curve

**Recommendation:** Start with Docker Compose for simplicity, migrate to Kubernetes when scaling demands require it.

### 2. EdgeX Foundry + UMH Core Integration Strategy

**Analysis:** UMH Core is indeed the right choice for the unified namespace aspect! Here's how EdgeX and UMH Core complement each other:

**EdgeX Foundry Role (Edge Layer):**
- **Device Connectivity**: Handles physical device connections and protocol translation
- **Device Services**: Manages device drivers for industrial protocols (OPC UA, Modbus, S7, etc.)
- **Local Processing**: Edge-side data validation and basic transformation
- **Device Management**: Device discovery, configuration, and health monitoring
- **Message Bus**: Local MQTT broker for edge device communication

**UMH Core Role (Unified Namespace Layer):**
- **Unified Namespace**: Provides the structured topic hierarchy (`umh.v1.enterprise.site.area.line._contract.tag`)
- **Data Modeling**: Validates and structures data using data contracts (`_raw`, `_pump_v1`, etc.)
- **Protocol Bridges**: 50+ industrial protocol connectors via Benthos-UMH
- **Stream Processing**: Real-time data transformation and aggregation
- **Cloud Integration**: Handles edge-to-cloud data flow and synchronization

**Integration Architecture:**

```
Factory Edge Box:
┌─────────────────────────────────────────────────────┐
│ EdgeX Foundry                                       │
│ ├── Device Services (OPC UA, Modbus, S7)          │
│ ├── Core Data (local storage/buffering)           │
│ ├── Core Metadata (device registry)               │
│ └── Message Bus (local MQTT)                      │
│                    │                               │
│ UMH Core                                           │
│ ├── Bridge (MQTT → UNS)                          │
│ ├── Data Models (_raw, _pump_v1)                 │
│ ├── Local UNS (buffering)                        │
│ └── Cloud Sync (to Hyperfactory)                 │
└─────────────────────────────────────────────────────┘
                     │
                     ▼ (MQTT/HTTP)
┌─────────────────────────────────────────────────────┐
│ Hyperfactory Cloud Instance                         │
│ ├── UMH Core (cloud instance)                     │
│ ├── TimescaleDB (time-series storage)             │
│ ├── PostgreSQL (relational data)                  │
│ └── Visualization/Analytics                        │
└─────────────────────────────────────────────────────┘
```

**Why This Combination Works:**
1. **Separation of Concerns**: EdgeX handles device complexity, UMH Core handles data organization
2. **Proven Pattern**: UMH Core already has bridges for industrial protocols
3. **Unified Namespace**: UMH Core's topic structure provides the organized data backbone
4. **Flexibility**: Can use EdgeX device services OR UMH Core bridges depending on requirements
5. **Scalability**: Both systems are designed for industrial scale

**Is EdgeX Necessary?**
- **For Simple Deployments**: UMH Core bridges alone might be sufficient
- **For Complex Device Management**: EdgeX provides better device lifecycle management
- **For Custom Protocols**: EdgeX device services offer more flexibility
- **For Edge Intelligence**: EdgeX provides better local processing capabilities

**Recommendation**: Start with UMH Core bridges for simplicity, add EdgeX when device management complexity requires it.

**Decision:** Use UMH Core for unified namespace + optionally EdgeX for complex device management

**Updated Understanding:**
- UMH Core provides the unified namespace and data organization we need
- EdgeX Foundry can complement UMH Core for complex device management scenarios
- Start with UMH Core bridges, add EdgeX when device complexity requires it

### 3. Multi-tenancy Strategy

**Separate Clusters Pros:**
- Complete isolation
- Independent scaling
- Easier compliance/security
- No noisy neighbor issues

**Separate Clusters Cons:**
- Higher infrastructure costs
- More operational complexity
- Resource waste for small customers

**Namespace Isolation Pros:**
- Resource efficiency
- Simpler operations
- Cost-effective for small customers
- Kubernetes-native approach

**Namespace Isolation Cons:**
- Potential security concerns
- Shared cluster dependencies
- More complex RBAC setup

**Database-level Isolation Pros:**
- Simplest to implement
- Lowest infrastructure cost
- Easy to start with

**Database-level Isolation Cons:**
- Weakest isolation
- Potential data leakage risks
- Scaling limitations

**Recommendation:** Start with namespace isolation, move to separate clusters for enterprise customers.

### 4. Message Broker Selection

**UMH Uses:** MQTT + Redpanda (Kafka-compatible)

**MQTT Broker Pros:**
- Lightweight, perfect for IoT/edge
- Low bandwidth usage
- Built-in QoS levels
- Industry standard for industrial IoT

**Redpanda Pros:**
- Kafka-compatible but simpler
- Better performance than Kafka
- Easier operations
- Good for cloud-side processing

**NATS Pros:**
- Extremely lightweight
- Built-in clustering
- Good for microservices

**Recommendation:** MQTT for edge-to-cloud communication + Redpanda for cloud-side message processing (following UMH pattern).

### 5. Database Strategy

**Time-series Database Pros:**
- Optimized for sensor data
- Better compression
- Built-in time-based queries
- Efficient for analytics

**PostgreSQL + TimescaleDB Pros:**
- Best of both worlds
- Relational data + time-series
- Single database to manage
- SQL familiarity

**Separate Databases:**
- PostgreSQL for customer/config data
- TimescaleDB/InfluxDB for sensor data

**Recommendation:** PostgreSQL + TimescaleDB extension for unified approach.

### 6. Data Retention Strategy

**Hot/Warm/Cold Storage Tiers:**
- **Hot (0-30 days)**: Fast SSD storage for real-time queries
- **Warm (30-365 days)**: Standard storage for historical analysis
- **Cold (1+ years)**: Object storage (S3-compatible) for compliance/archival

**Implementation:**
- Automatic data lifecycle policies
- Compression for older data
- Configurable retention per customer

### 7. Edge-to-Cloud Synchronization Strategies

**On-premise Cache Options:**
- **Local TimescaleDB**: Buffer data locally during outages
- **Redis**: Fast cache for recent data
- **File-based buffering**: Simple append-only logs

**Synchronization Strategies:**
- **Store-and-forward**: Queue data during outages, sync when connected
- **Delta sync**: Only send changes since last successful sync
- **Compression**: Reduce bandwidth usage
- **Prioritization**: Critical data first, historical data later

**Recommended Approach:**
- Local TimescaleDB cache on edge
- MQTT with QoS 1 for reliable delivery
- Automatic retry with exponential backoff
- Data compression for large transfers

# Architecture & Infrastructure

## Decided

- **Cloud Platform**: Hyperfactory instances will run in the cloud on OpenMetal infrastructure
- **Edge Technology**: UMH Core (single container) for edge device connectivity and unified namespace
- **Container Orchestration**: Docker Compose for simplicity, migrate to Kubernetes when scaling demands require it
- **Data Flow**: Edge devices stream data to hyperfactory instances running in the cloud
- **Infrastructure Context**: There is existing infrastructure code in the /context folder from another project that controls a Kubernetes cluster, which may or may not be used for this project
- **Architecture Philosophy**: Keep things simple for small team/product, but maintain ability to scale when necessary
- **Scaling Priority**: Factory data flow is relatively steady, so immediate elasticity is not the highest priority
- **EdgeX Foundry Decision**: Not using EdgeX Foundry - UMH Core provides sufficient protocol connectivity (50+ industrial connectors) without the operational complexity of microservices architecture

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

### 2. UMH Core Architecture (EdgeX Foundry Removed)

**Decision:** Use UMH Core only - EdgeX Foundry adds unnecessary complexity without significant benefits for our use case.

**UMH Core Role (Complete Edge Solution):**
- **Unified Namespace**: Provides the structured topic hierarchy (`umh.v1.enterprise.site.area.line._contract.tag`)
- **Protocol Connectivity**: 50+ industrial protocol connectors via Benthos-UMH (OPC UA, Modbus, S7, Ethernet/IP, etc.)
- **Data Modeling**: Validates and structures data using data contracts (`_raw`, `_pump_v1`, etc.)
- **Stream Processing**: Real-time data transformation and aggregation
- **Local Buffering**: Embedded Redpanda for offline capability
- **Cloud Integration**: Handles edge-to-cloud data flow and synchronization

**Simplified Architecture:**

```
Factory Edge Box:
┌─────────────────────────────────────────────────────┐
│ UMH Core (Single Container)                        │
│ ├── Protocol Bridges (OPC UA, Modbus, S7, etc.)   │
│ ├── Unified Namespace (structured topics)          │
│ ├── Data Contracts (_raw, _pump_v1)               │
│ ├── Redpanda (local buffering)                    │
│ ├── Benthos-UMH (stream processing)               │
│ └── Cloud Sync (to Hyperfactory)                  │
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

**Why UMH Core Alone is Sufficient:**
1. **Single Container Simplicity**: Aligns with our "keep it simple" philosophy
2. **Complete Protocol Coverage**: 50+ industrial connectors cover factory needs
3. **Built-in Unified Namespace**: No additional integration required
4. **Operational Simplicity**: One container vs. 6+ EdgeX microservices
5. **Purpose-Built**: Designed specifically for industrial IoT unified namespace patterns

**EdgeX Foundry Analysis - Why We Don't Need It:**
- **Microservices Complexity**: 6+ containers vs. 1 container
- **Redundant Protocol Support**: UMH Core already covers needed protocols
- **Over-Engineering**: Device lifecycle management features are overkill for factory scenarios
- **Operational Overhead**: Multiple databases, configurations, and services to manage

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

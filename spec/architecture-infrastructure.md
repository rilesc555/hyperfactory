# Architecture & Infrastructure

## Decided

### Architecture Overview

- **Cloud Platform**: OpenMetal infrastructure for hyperfactory instances
- **Edge Technology**: UMH Core + EMQX MQTT broker for comprehensive device connectivity
- **Container Orchestration**: Docker Compose for both edge and cloud deployments
- **Data Flow**: Edge devices → UMH Core/EMQX → Ziti tunnel → Customer cloud instances
- **Message Broker**: EMQX (edge MQTT) + UMH Core embedded Redpanda (unified namespace)
- **Database Strategy**: PostgreSQL + TimescaleDB extension for unified relational and time-series data
- **Multi-tenancy**: Per-customer Docker Compose instances (separate containers per customer)
- **Architecture Philosophy**: Keep things simple for small team/product, but maintain ability to scale when necessary


### 1. Container Orchestration Decision: Docker Compose

**Decision:** Use Docker Compose for both edge and cloud deployments.

**Why Docker Compose:**
- **Simplicity**: Perfect for small team and focused product
- **Fast Development**: Quick iteration and deployment cycles
- **Minimal Overhead**: No complex orchestration layer to manage
- **Predictable Workloads**: Factory data flows are steady, not highly elastic
- **Easy Operations**: Simple to understand, deploy, and troubleshoot
- **Multi-Host Support**: Docker Swarm mode available if needed later

**Edge Deployment:**
- UMH Core + EMQX in single docker-compose.yml
- Simple deployment to factory edge boxes
- Easy to replicate across multiple sites

**Cloud Deployment:**
- Per-customer instances using docker-compose
- TimescaleDB, PostgreSQL, Grafana, UMH Core, Odoo agent
- Simple scaling by adding more customer instances

**Future Considerations:**
- Can migrate to Kubernetes later if scaling demands require it
- Existing infrastructure code in /context folder available if needed
- Docker Compose provides good foundation for eventual K8s migration


### 2. Edge Technology: UMH Core + EMQX Architecture

**Decision:** Use UMH Core as primary edge solution with EMQX MQTT broker for comprehensive device connectivity.

**UMH Core Role (Primary Edge Solution):**
- **Unified Namespace**: Provides structured topic hierarchy (`umh.v1.enterprise.site.area.line._contract.tag`)
- **Protocol Connectivity**: 50+ industrial protocol connectors via Benthos-UMH (OPC UA, Modbus, S7, Ethernet/IP, etc.)
- **Data Modeling**: Validates and structures data using data contracts (`_raw`, `_pump_v1`, etc.)
- **Stream Processing**: Real-time data transformation and aggregation
- **Local Buffering**: Embedded Redpanda for offline capability
- **Cloud Integration**: Handles edge-to-cloud data flow and synchronization

**EMQX MQTT Broker Role (Auxiliary Edge Component):**
- **MQTT Device Support**: Handles devices that only support MQTT protocol
- **Local MQTT Hub**: Provides reliable MQTT broker for legacy sensors and devices
- **Bridge to UMH Core**: MQTT topics are bridged into UMH Core's unified namespace
- **Production Ready**: Built-in clustering, persistence, monitoring, and web dashboard

**Why This Architecture Works:**
- **Hybrid Simplicity**: UMH Core handles most protocols directly, EMQX covers MQTT gap
- **Complete Protocol Coverage**: 50+ industrial connectors + MQTT broker covers all factory needs
- **Built-in Unified Namespace**: All data flows into UMH Core's structured topics
- **Operational Simplicity**: Two lightweight containers with simple configuration
- **Purpose-Built**: Designed specifically for industrial IoT unified namespace patterns
- **Flexible**: Can handle both modern industrial protocols and legacy MQTT devices

### 3. Multi-tenancy Strategy: Per-Customer OpenMetal Projects

**Decision:** Use separate OpenMetal projects per customer with Docker Compose deployments and self-service scaling.

**Why Per-Customer OpenMetal Projects:**
- **Complete Infrastructure Isolation**: Each customer gets their own OpenMetal project with dedicated networking, compute, and storage
- **Security Boundaries**: Network-level isolation between customers eliminates data leakage risks
- **Billing Separation**: Clear cost allocation and real-time billing integration with Odoo
- **Resource Control**: Can set quotas and limits per project, with customer self-service scaling
- **Compliance**: Easier to meet customer-specific regulatory requirements
- **Operational Simplicity**: Docker Compose within each project vs. complex K8s multi-tenancy
- **Customization**: Can customize entire stack per customer (branding, features, integrations)

**Architecture Overview:**
```
Hyperfactory Control Plane (OpenMetal)
├── Control API (customer management, billing)
├── Infrastructure Orchestrator (OpenMetal API integration)
├── Global Monitoring (resource usage, health)
└── Customer Registry (accounts, limits, billing)

Customer Project A (OpenMetal)
├── Docker Compose Stack
│   ├── UMH Core (cloud instance)
│   ├── TimescaleDB + PostgreSQL
│   ├── Grafana + Customer Dashboard
│   └── Odoo Agent + Ziti Controller
└── Hyperfactory Agent (monitoring, remote management)

Customer Project B (OpenMetal)
├── Docker Compose Stack
└── Hyperfactory Agent
```

**Customer Self-Service Scaling:**
- **Dashboard Integration**: Scaling controls embedded in customer dashboard
- **Real-time Billing**: Odoo API integration for immediate cost calculation
- **Resource Limits**: Configurable per-customer limits to prevent runaway costs
- **Automated Provisioning**: Payment triggers automatic resource allocation
- **Sandbox Option**: Trial instances for customer evaluation

**Implementation Benefits:**
- **Customer Autonomy**: Self-service scaling within defined limits
- **Operational Control**: Centralized management with per-customer isolation
- **Cost Transparency**: Real-time resource tracking and billing
- **Simple Operations**: Docker Compose easier than K8s multi-tenancy
- **Scalable Growth**: Can handle many customers with consistent patterns

**Resource Management:**
- **Capacity Monitoring**: Track OpenMetal resource usage to prevent over-allocation
- **Automated Alerts**: Notify when approaching capacity limits
- **Scaling Validation**: Check available resources before customer scaling requests
- **Billing Integration**: Real-time cost calculation via Odoo payment system

### 4. Control Plane Architecture

**Hyperfactory Control Plane Components:**

**Control API Service:**
- **Customer Management**: CRUD operations for customer accounts
- **Resource Management**: Handle scaling requests and validation
- **Billing Integration**: Real-time Odoo API integration for payments
- **Authentication**: Customer dashboard authentication and authorization
- **Rate Limiting**: Prevent abuse of scaling operations

**Infrastructure Orchestrator:**
- **OpenMetal API Integration**: Provision and manage customer projects
- **Docker Compose Templates**: Standardized deployment templates
- **Resource Provisioning**: Automated customer environment setup
- **Scaling Operations**: Handle customer self-service scaling requests
- **Health Monitoring**: Track customer environment health

**Customer Lifecycle Management:**
```
Customer Journey:
1. Trial Signup → Sandbox Environment (limited resources, 14-day trial)
2. Payment → Full Environment Provisioning (automated via Odoo)
3. Self-Service Scaling → Resource adjustments within limits
4. Monitoring → Continuous health and usage tracking
5. Billing → Real-time cost calculation and payment processing
```

**Capacity Management:**
- **OpenMetal Resource Tracking**: Monitor total available vs. allocated resources
- **Predictive Scaling**: Alert when approaching capacity limits (e.g., 80% utilization)
- **Customer Queuing**: Queue provisioning requests when at capacity
- **Auto-scaling**: Automatically provision additional OpenMetal resources when needed

**Customer Agent (deployed in each customer project):**
- **Resource Reporting**: Send metrics back to control plane
- **Health Monitoring**: Container health, service status, performance metrics
- **Remote Management**: Receive and execute scaling commands
- **Security**: Secure communication channel back to control plane

**Implementation Considerations:**
- **Control Plane Hosting**: Runs in dedicated OpenMetal project for Hyperfactory operations
- **High Availability**: Control plane should be resilient to failures
- **Backup Strategy**: Regular backups of customer registry and configuration
- **Disaster Recovery**: Plan for control plane and customer data recovery

### 5. Edge-to-Cloud Communication: Redpanda Connect Core-to-Core Bridge

**Decision:** Replace the MQTT bridge with a Redpanda Connect (Benthos-UMH) one-way bridge that reads from the edge UMH Core’s Redpanda topics and writes to the cloud UMH Core’s Redpanda topics. Direction is edge → cloud only.

**Why Redpanda Connect Bridge:**
- **At-least-once delivery**: Consumer offsets are committed only after the cloud write is acknowledged
- **Ordering by key**: Per-device/per-signal ordering preserved by using message keys
- **Simple operations**: Runs alongside UMH on the edge; no extra broker(s) required
- **UMH-aligned**: Suggested interim approach while UMH core-to-core plugin is in development
- **Efficient**: Batching + compression yields high throughput

**Architecture Flow:**
```
Edge UMH Core (Redpanda) → Redpanda Connect (Edge) → [TLS/mTLS] → Cloud UMH Core (Redpanda)
```

**Edge Bridge (minimal example):**
```yaml
input:
  redpanda:
    seed_brokers: ["redpanda:9092"]           # edge cluster
    topics: ["umh.v1.telemetry.*"]            # or your UNS topics
    regex_topics: true
    consumer_group: "bridge-edge-to-cloud"
    start_from_oldest: true
pipeline:
  processors:
    - mapping: |
        root = this
        meta.edge_id = env("EDGE_ID")
output:
  redpanda:
    seed_brokers: ["cloud-redpanda:9092"]     # cloud cluster
    topic: ${! meta("topic") }                 # mirror topic name
    key: ${! coalesce(meta("key"), concat(json("device_id"), ":", json("signal"))) }
    tls:
      enabled: true
      root_cas_file: /etc/ssl/certs/ca.pem
```

**Security & Reliability:**
- **TLS/mTLS** between edge and cloud Redpanda clusters (per-customer CA)
- **Offset commits after ack**: preserves at-least-once semantics end-to-end
- **Batching & retries**: producer retries with backoff; bounded batch size/time window
- **Ordering**: stable keys (e.g., `device_id:signal`) ensure per-key ordering

**Benefits:**
- **Unified bus**: Same protocol and tooling on edge and cloud
- **Operational simplicity**: No MQTT bridge to run/monitor
- **Scalable**: Efficient for high-rate telemetry
- **Ready today**: Matches UMH team guidance for inter-core sync


### 6. Message Broker Selection

**UMH Uses:** MQTT + Redpanda (Kafka-compatible)

**MQTT Broker Pros:**
- Lightweight, perfect for IoT/edge device connectors
- Low bandwidth usage
- Built-in QoS levels
- Industry standard for industrial IoT

**Redpanda Pros:**
- Kafka-compatible but simpler to operate
- High throughput and low latency
- Unified protocol on edge and cloud
- Great for stream processing and durable buffers

**Recommendation:** Use a Redpanda Connect core-to-core bridge for edge→cloud data movement (one-way), keep MQTT for device-facing connectors, and use Redpanda for stream processing on both edge and cloud.

### 7. Database Strategy: PostgreSQL + TimescaleDB

**Decision:** Use PostgreSQL with TimescaleDB extension for unified relational and time-series data storage.

**Why PostgreSQL + TimescaleDB:**
- **Unified Data Model**: Single database handles both relational (customers, configurations, metadata) and time-series (sensor data, metrics) data
- **SQL Familiarity**: Standard PostgreSQL SQL interface for all data operations
- **Operational Simplicity**: One database system to manage, backup, and monitor
- **Performance**: TimescaleDB provides time-series optimizations while maintaining PostgreSQL compatibility
- **Ecosystem**: Rich PostgreSQL ecosystem (tools, extensions, monitoring, backup solutions)
- **Scaling**: Horizontal scaling available through TimescaleDB's distributed hypertables
- **Cost Effective**: No need for separate database licenses or specialized expertise

**Data Organization:**

**Relational Tables (Standard PostgreSQL):**
```sql
-- Customer and configuration data
customers (id, name, subscription_tier, created_at, settings)
sites (id, customer_id, name, location, timezone, config)
devices (id, site_id, device_type, protocol, connection_config)
users (id, customer_id, email, role, permissions)
alerts (id, customer_id, rule_config, notification_config)
```

**Time-Series Tables (TimescaleDB Hypertables):**
```sql
-- Sensor data with automatic partitioning by time
sensor_data (time, device_id, tag_name, value, quality, metadata)
system_metrics (time, customer_id, metric_name, value, labels)
audit_logs (time, customer_id, user_id, action, resource, details)
```

**TimescaleDB Features Used:**
- **Hypertables**: Automatic time-based partitioning for sensor_data tables
- **Compression**: Automatic compression for older data (configurable per customer)
- **Continuous Aggregates**: Pre-computed rollups for common analytics queries
- **Data Retention Policies**: Automatic cleanup of old data based on customer settings
- **Parallel Query**: Faster analytics across time ranges

**Per-Customer Database Strategy:**
- **Shared Schema**: All customers use same database schema for operational simplicity
- **Row-Level Security**: PostgreSQL RLS ensures customer data isolation
- **Customer ID Partitioning**: TimescaleDB partitions include customer_id for performance
- **Backup Isolation**: Per-customer backup and restore capabilities

**Performance Optimizations:**
```sql
-- Hypertable with customer-aware partitioning
SELECT create_hypertable('sensor_data', 'time',
  partitioning_column => 'customer_id',
  number_partitions => 4);

-- Compression policy (compress data older than 7 days)
SELECT add_compression_policy('sensor_data', INTERVAL '7 days');

-- Retention policy (delete data older than customer's retention setting)
SELECT add_retention_policy('sensor_data', INTERVAL '2 years');

-- Continuous aggregate for hourly rollups
CREATE MATERIALIZED VIEW sensor_data_hourly
WITH (timescaledb.continuous) AS
SELECT time_bucket('1 hour', time) AS hour,
       device_id, tag_name,
       avg(value) as avg_value,
       min(value) as min_value,
       max(value) as max_value,
       count(*) as sample_count
FROM sensor_data
GROUP BY hour, device_id, tag_name;
```

**Benefits for Hyperfactory:**
- **Developer Productivity**: Single SQL interface for all data operations
- **Operational Simplicity**: One database system to manage across all customers
- **Cost Efficiency**: No separate time-series database licensing or infrastructure
- **Flexibility**: Can handle both structured business data and high-frequency sensor data
- **Ecosystem Integration**: Works with existing PostgreSQL tools (pgAdmin, monitoring, backup)
- **Future-Proof**: Can scale from single-node to distributed as customer base grows

### 8. Data Retention Strategy: Single-Tier Storage with Growth Path

**Decision:** Start with single-tier storage using OpenMetal's standard Ceph block storage, with TimescaleDB compression and retention policies for cost optimization.

**Why This Approach:**
- **Operational Simplicity**: Single storage tier eliminates complexity of data movement and multiple storage classes
- **Cost Effective**: TimescaleDB compression provides 5-10x storage reduction without infrastructure complexity
- **Growth Ready**: Can easily add storage tiers later when customer base and data volumes justify the complexity
- **Proven Technology**: Standard Ceph block storage with PostgreSQL is well-understood and reliable
- **Fast Implementation**: No need to configure multiple storage pools or lifecycle policies

**Single-Tier Implementation:**

**Primary Storage: OpenMetal Ceph Block Storage**
- **Hardware**: Standard OpenMetal storage nodes with mixed SSD/HDD configuration
- **Performance**: Good balance of performance and cost for all data access patterns
- **Ceph Pool**: Single replicated pool (3x replication) for reliability
- **TimescaleDB**: All data on same storage with compression for cost optimization

**Data Lifecycle Management:**

**TimescaleDB Compression and Retention:**
```sql
-- Compression policy: compress data older than 1 day
SELECT add_compression_policy('sensor_data', INTERVAL '1 day');

-- Retention policy: delete data older than configurable period
SELECT add_retention_policy('sensor_data', INTERVAL '2 years');

-- Per-customer retention (configurable)
SELECT add_retention_policy('customer_123_sensor_data', INTERVAL '5 years');
```

**Storage Configuration:**
```yaml
# Single storage class for all data
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-block
parameters:
  pool: rbd-pool
  replication: "3"
  fsType: ext4
```

**Per-Customer Configuration:**
```yaml
# Customer retention settings
customer_retention:
  standard_retention_years: 2    # Standard customers: 2 years
  premium_retention_years: 5     # Premium customers: 5 years
  compression_days: 1            # Compress after 1 day

# Configurable per customer
customer_settings:
  customer_123:
    retention_years: 7           # Custom retention for compliance
    compression_days: 1
```

**Cost Optimization Through Compression:**

**TimescaleDB Compression Benefits:**
- **Storage Reduction**: 5-10x compression for time-series data
- **Query Performance**: Compressed data still queryable with good performance
- **Automatic Management**: Compression happens automatically based on policies
- **Cost Savings**: Significant storage cost reduction without infrastructure complexity

**Performance Characteristics:**
- **Recent Data**: Fast queries on uncompressed data (last 24 hours)
- **Historical Data**: Good performance on compressed data with slight latency increase
- **Compression Ratio**: Typical 8x compression for sensor data
- **Query Impact**: 10-20% performance impact on compressed data vs. uncompressed

**Backup and Disaster Recovery:**
- **Primary Protection**: Ceph 3x replication across OpenMetal nodes
- **Backup Strategy**: Regular PostgreSQL dumps to OpenMetal object storage
- **Point-in-Time Recovery**: WAL archiving for transaction-level recovery
- **Cross-Region**: Optional backup replication to secondary OpenMetal region

**Growth Path to Multi-Tier Storage:**

**When to Add Storage Tiers:**
- **Data Volume**: When total storage exceeds 10-20TB per customer
- **Cost Pressure**: When storage costs become significant portion of customer bills
- **Performance Requirements**: When query performance on old data becomes issue
- **Customer Demand**: When customers request different retention/performance tiers

**Easy Migration Path:**
```sql
-- Future: Add warm storage tablespace
CREATE TABLESPACE warm_storage LOCATION '/mnt/warm-storage';

-- Future: Move old chunks to warm storage
SELECT add_move_chunk_policy('sensor_data',
  older_than => INTERVAL '30 days',
  destination_tablespace => 'warm_storage');
```

**Implementation Benefits:**
- **Simple Operations**: Single storage system to monitor and maintain
- **Fast Deployment**: No complex storage tier configuration required
- **Cost Effective**: Compression provides immediate cost benefits
- **Reliable**: Well-tested PostgreSQL + Ceph combination
- **Scalable**: Can easily add complexity when business justifies it
- **Customer Flexibility**: Configurable retention periods per customer needs

### 9. Edge-to-Cloud Synchronization Strategy: Redpanda Connect Core-to-Core Bridge

**Decision:** Run a Redpanda Connect (Benthos-UMH) bridge on the edge that reads from the edge UMH Core’s Redpanda topics and writes to the cloud UMH Core’s Redpanda topics. One-way replication (edge → cloud) with at-least-once delivery and idempotent cloud ingestion.

**Why This Approach:**
- **Offline-first**: Uses edge Redpanda as the durable buffer; replays on reconnect
- **Simple Operations**: No MQTT bridge required; single protocol end-to-end
- **At-least-once Delivery**: Consumer offsets are committed after cloud produce acks
- **Idempotent Ingestion**: Cloud deduplication guarantees no double inserts
- **UMH-aligned**: Matches UMH guidance while their core-to-core plugin is under development

**End-to-End Flow:**
1) Devices → UMH connectors → UMH Unified Namespace (edge Redpanda topics)
2) Edge Redpanda retains messages by age/size (e.g., 3 days or 20 GiB)
3) Redpanda Connect (edge) consumes, batches, and produces to cloud Redpanda over TLS/mTLS
4) Cloud ingestion service reads cloud topics and upserts into TimescaleDB

**Message Envelope (JSON):**
- `event_id`: UUIDv7 (unique per message)
- `device_id`: canonical device identifier
- `seq`: monotonically increasing per device publisher
- `ts`: measurement timestamp (edge clock)
- `ingest_ts`: set by cloud on write
- `schema_version`: starts at `1`
- `payload`: typed values (number/string/boolean)

**Delivery Semantics:**
- **Offset commits after ack**: at-least-once from edge → cloud
- **Ordering**: Use message keys (e.g., `device_id:signal`) for per-key ordering
- **Retry**: Producer retries with backoff on transient errors

**Edge Buffering & Replay:**
- **Primary buffer**: UMH Core embedded Redpanda topics (default: 3 days or 20 GiB)
- **Replay**: On reconnect, bridge drains from oldest committed offset per partition

**Batching & Compression:**
- Batch by time window (e.g., 1s) or count (e.g., 500 messages)
- Enable producer compression (e.g., lz4) for throughput and cost efficiency
- Cloud ingestor expands and upserts in batches

**Ordering & Idempotency:**
- Ordering guaranteed within a partition (per key)
- Cloud upsert key: `(device_id, ts, seq)` or `event_id`
- On conflict: do nothing (dedupe) to satisfy at-least-once

**Topic Namespace (edge → cloud):**
- Prefer mirroring topic names between cores (e.g., `umh.v1.telemetry.*`)
- Use regex input on edge bridge; map to the same topic name on cloud

**Security:**
- mTLS between edge and cloud Redpanda clusters (per-customer CA and client certs)
- TLS 1.2+ only; strong cipher suites
- ACLs per tenant/topic prefix

**Operational Controls:**
- Backpressure: bounded batches; observe producer acks and throttle on lag
- Monitoring: consumer lag, batch size, produce latency, error/retry rates

**Minimal Config Excerpts:**

Edge bridge (Redpanda Connect):
```yaml
input:
  redpanda:
    seed_brokers: ["redpanda:9092"]
    topics: ["umh.v1.telemetry.*"]
    regex_topics: true
    consumer_group: "bridge-edge-to-cloud"
    start_from_oldest: true
output:
  redpanda:
    seed_brokers: ["cloud-redpanda:9092"]
    topic: ${! meta("topic") }
    key: ${! coalesce(meta("key"), concat(json("device_id"), ":", json("signal"))) }
    tls:
      enabled: true
      root_cas_file: /etc/ssl/certs/ca.pem
```

Cloud ingestion (idempotent upsert):
```sql
INSERT INTO sensor_data(device_id, ts, seq, payload)
VALUES ($1, $2, $3, $4)
ON CONFLICT (device_id, ts, seq) DO NOTHING;
```

**Defaults (can be tuned per customer):**
- Buffer retention: 3 days or 20 GiB (whichever first)
- Max batch: 500 msgs or 1s window
- Compression: lz4 on producer

**Benefits:**
- Unified protocol and tooling across edge and cloud
- Strong delivery guarantees with simple components
- Clear dedupe model avoids duplicates in TimescaleDB
- Compose-friendly and easy to monitor

## Undecided

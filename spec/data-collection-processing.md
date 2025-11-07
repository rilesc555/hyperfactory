# Data Collection & Processing

## Decided

- **Data Sources**: Hyperfactory collects, processes, and analyzes data from various factory sources
- **Unified Namespace**: Provides a unified namespace for all factory data
- **Edge Collection**: Uses EdgeX Foundry (or similar) to connect to PLCs, sensors, and other devices
- **Data Flow**: Edge devices stream data to hyperfactory instances in the cloud

## Undecided

1. What industrial protocols must we support? (OPC UA, Modbus TCP/RTU, EtherNet/IP, PROFINET, S7, MTConnect, etc.)
2. How do we handle protocol discovery and auto-configuration?
3. What's our data model for the unified namespace? (ISA-95, custom, UMH-style?)
4. How do we handle data transformation and normalization at the edge vs. cloud?
5. What's our approach to real-time vs. batch processing?
6. How do we handle data quality, validation, and anomaly detection?

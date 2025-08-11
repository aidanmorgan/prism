# Prism: Serverless OTel Analytics Platform

**Product Overview & Requirements Document**

## Product Vision

Prism is a serverless, columnar telemetry analytics platform optimized for OpenTelemetry (OTel) data ingestion, storage, and querying. The platform provides low-latency, high-throughput processing of logs, traces, and metrics with multi-tenant isolation and global scale capabilities.

## Core Value Proposition

- **Serverless-first**: Zero infrastructure management with automatic scaling
- **OTel-native**: Purpose-built for OpenTelemetry data patterns
- **Multi-tenant**: Secure isolation with transparent sharding
- **Global scale**: Multi-region deployment with data residency controls
- **Cost-effective**: Tiered storage with intelligent lifecycle management

## Product Objectives

### Primary Objectives

1. **High-Performance Ingest**
   - Low-latency, high-throughput ingest for OTel data (logs, traces, metrics)
   - Support for OTLP/HTTP and OTLP/gRPC protocols
   - Eventized row processing for all signal types

2. **Columnar Storage Architecture**
   - Append-only hot store with time-boxed segments
   - Atomic appends using NFS symlink locks
   - Hot/warm/cold tiering: EFS → S3 blocks → Parquet

3. **Serverless Query Engine**
   - Lambda-based query service
   - Zipkin-compatible APIs for seamless migration
   - Custom JSON DSL for advanced analytics
   - Hybrid planner reading both hot EFS blocks and cold Parquet

4. **Multi-Tenancy & Sharding**
   - Complete tenant isolation with transparent sharding
   - Automatic hotspot detection and rebalancing
   - Zero downtime shard migrations

5. **Data Residency & Compliance**
   - Tenants pinned to home regions
   - Configurable data residency policies (strict/permitted/deferred)
   - Optional secondary regions for disaster recovery

6. **Minimal Indexing Strategy**
   - Time partitioning as primary organization
   - Per-block statistics and small dictionaries
   - Optional Bloom/Cuckoo filters
   - Parallel fan-out query execution

### Non-Goals

- **Not a general OLAP warehouse**: Optimized specifically for telemetry patterns
- **Not for real-time alerting**: Optimized for analytical queries over historical data
- **Not for transactional workloads**: Append-only, scan-heavy access patterns

## Target Users

### Primary Users

1. **DevOps/SRE Teams**
   - Need observability data for troubleshooting and monitoring
   - Require fast trace lookups and service dependency analysis
   - Value cost-effective long-term storage

2. **Platform Engineering Teams**
   - Managing observability infrastructure at scale
   - Need multi-tenant capabilities for organization isolation
   - Require global deployment with data residency controls

3. **Application Developers**
   - Debugging distributed applications
   - Performance analysis and optimization
   - Understanding service interactions and dependencies

### Secondary Users

1. **Security Teams**
   - Audit trail analysis from telemetry data
   - Anomaly detection in application behavior

2. **Business Analytics Teams**
   - Application performance impact on business metrics
   - Usage pattern analysis

## Functional Requirements

### Ingest Requirements

- **Protocol Support**: OTLP/HTTP (primary), OTLP/gRPC, Zipkin v2 compatibility
- **Throughput**: Support for high-volume telemetry ingestion (target: 100k+ events/second per tenant)
- **Latency**: Ingest p99 < 500ms per batch
- **Batching**: Support 2-16k records/request with min body size 256-512 KB
- **Context Preservation**: Full W3C baggage and tracestate handling
- **Idempotency**: Protection against duplicate ingestion

### Storage Requirements

- **Hot Storage**: EFS-based columnar storage with 5-minute segments
- **Lateness Handling**: 30-minute lateness window with delta segments
- **Compression**: Zstd level 1-3 for hot storage, optimized Parquet for cold
- **Retention**: Configurable hot/warm/cold lifecycle policies
- **Durability**: Multi-AZ storage with optional cross-region replication

### Query Requirements

- **API Compatibility**: Full Zipkin v2 API compatibility
- **Query Language**: JSON DSL supporting complex filters and aggregations
- **Performance**: Query p95 < 2s for last 1 hour of data
- **Scale**: Support queries across TB-scale datasets
- **Real-time**: Hot data visibility < 2 seconds after ingest

### Multi-Tenancy Requirements

- **Isolation**: Complete data and compute isolation between tenants
- **Transparency**: Tenants unaware of sharding or other tenants
- **Scaling**: Automatic shard rebalancing based on usage patterns
- **Fairness**: Resource allocation fairness across tenants

### Security Requirements

- **Authentication**: API key-based authentication
- **Authorization**: Tenant-scoped access controls
- **Encryption**: TLS in transit, KMS encryption at rest
- **Compliance**: Data residency controls, audit logging
- **Network Security**: VPC isolation, security group controls

## Non-Functional Requirements

### Performance

- **Ingest Latency**: p99 < 500ms
- **Query Latency**: p95 < 2s for recent data (1 hour)
- **Hot Visibility**: < 2s from ingest to query availability
- **Throughput**: 100k+ events/second per tenant sustained

### Availability

- **Uptime**: 99.9% availability SLA
- **Multi-AZ**: Automatic failover within region
- **Disaster Recovery**: RTO < 4 hours, RPO < 1 hour for critical tenants

### Scalability

- **Horizontal Scaling**: Automatic Lambda scaling
- **Storage Scaling**: Elastic storage with lifecycle management
- **Tenant Scaling**: Support 1000+ tenants per region
- **Data Volume**: PB-scale data per tenant capability

### Cost Optimization

- **Tiered Storage**: Automatic migration to cost-effective storage tiers
- **Query Optimization**: Bytes-scanned minimization through pruning
- **Resource Efficiency**: Graviton processors, optimized memory allocation
- **Lifecycle Management**: Automated data archival and deletion

## Success Metrics

### Technical Metrics

- **Ingest Performance**: Sustained throughput, latency percentiles
- **Query Performance**: Response times, bytes scanned efficiency
- **Storage Efficiency**: Compression ratios, cost per GB stored
- **Availability**: Uptime, error rates, mean time to recovery

### Business Metrics

- **Customer Satisfaction**: Query response times, data freshness
- **Cost Efficiency**: Storage cost per tenant, compute cost optimization
- **Adoption**: API usage growth, tenant onboarding rate
- **Reliability**: Incident frequency, data consistency SLAs

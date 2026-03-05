---
name: cross-orchestrator
version: "2.4.1"
description: AI-powered orchestrator for managing infrastructure across multiple container platforms
maintainer: infrastructure@crossorch.io
tags:
  - infrastructure
  - ai
  - automation
  - kubernetes
  - docker
  - nomad
  - multi-cluster
requires:
  - kubectl >= 1.28
  - docker >= 24.0
  - nomad >= 1.6.0
  - python >= 3.10
  - tensorflow-serving >= 2.12
---

# Cross Orchestrator Skill

## Purpose

Cross Orchestrator provides AI-driven infrastructure orchestration across heterogeneous container platforms. It eliminates platform silos by abstracting Kubernetes, Docker Swarm, and Nomad into a unified control plane. The skill uses machine learning to optimize resource allocation, predict scaling needs, and automate cross-cluster failover.

### Real Use Cases

1. **Hybrid Cloud Management**: Deploy and manage applications across AWS EKS, Azure AKS, Google GKE, and on-premise Kubernetes clusters using identical commands
2. **Cost Optimization**: Automatically shift workloads between spot and on-demand instances based on AI-predicted demand patterns
3. **Disaster Recovery**: Configure active-active multi-region deployments with AI-driven traffic routing and automatic failover during azure/aws zone outages
4. **CI/CD Pipeline Acceleration**: Use AI to determine optimal deployment windows and canary analysis across production clusters
5. **Edge Computing**: Orchestrate container workloads across 1000+ edge nodes with adaptive throttling based on network conditions

## Scope

This skill provides complete orchestration capabilities for:

### Command Categories

**Deployment Operations:**
```
cross-orch deploy <service> --platform <k8s|docker|nomad> [--replicas <n>] [--resources <cpu:mem>] [--image <tag>] [--clusters <list>]
cross-orch rollback <deployment-id> [--to <version>] [--clusters <list>]
cross-orch upgrade <service> <new-image> [--strategy <rolling|bluegreen>] [--health-timeout <seconds>]
```

**Scaling Operations:**
```
cross-orch scale <service> <replicas|auto> [--clusters <list>] [--respect-quotas]
cross-orch autoscale <service> [--cpu-threshold <percent>] [--memory-threshold <percent>] [--cooldown <minutes>]
cross-orch predict-scale <service> --window <hours> [--confidence <percent>]
```

**Configuration Management:**
```
cross-orch config sync <config-file> [--clusters <list>] [--dry-run] [--validate]
cross-orch config diff <service> [--clusters <list>]
cross-orch config push <config-file> [--clusters <list>] [--rollout]
cross-orch config rollback <config-revision> [--clusters <list>]
```

**Health & Monitoring:**
```
cross-orch health <service> [--cluster <name>] [--deep] [--timeout <seconds>]
cross-orch metrics <service> [--range <duration>] [--percentiles <p50,p95,p99>]
cross-orch alerts list [--active] [--severity <level>]
cross-orch alerts acknowledge <alert-id>
```

**Multi-Cluster Operations:**
```
cross-orch cluster add <name> <platform> <kubeconfig|endpoint> [--region <region>]
cross-orch cluster remove <name> [--force]
cross-orch cluster status [--detail]
cross-orch cluster failover <service> --to <target-cluster> [--grace-period <seconds>]
```

**AI Optimization:**
```
cross-orch ai optimize <service> [--objective <cost|performance|availability>] [--budget <amount>]
cross-orch ai recommendations <service> [--limit <n>]
cross-orch ai simulate <service> --replicas <n> --duration <minutes>
```

**Infrastructure Provisioning:**
```
cross-orch infra create <template-name> [--count <n>] [--region <region>] [--tags <key:value>]
cross-orch infra destroy <resource-id> [--force] [--wait]
cross-orch infra list [--cluster <name>] [--type <vm|network|storage>]
```

## Work Process

### Pre-Execution Phase
1. **Intent Parsing**: AI model analyzes natural language input to extract:
   - Target services/resources
   - Desired state/intent
   - Implicit constraints (time windows, budget limits)
   - Platform preferences

2. **Dependency Resolution**: Builds directed acyclic graph of:
   - Service dependencies (database → app → cache)
   - Platform requirements (GPU nodes, specific OS)
   - Resource dependencies (PVCs, config maps, secrets)

3. **Strategy Selection**: Determines optimal approach:
   - Deployment strategy (rolling, blue-green, canary)
   - Cluster selection based on capacity, latency, cost
   - Resource allocation using AI-predicted usage patterns

4. **Impact Analysis**: Simulates outcomes:
   - Risk assessment (service disruption probability)
   - Resource utilization forecasts
   - Cost implications
   - Rollback complexity

### Execution Phase

**Step 1: Validation**
```bash
# Verify cluster connectivity
cross-orch cluster ping all --timeout 10s
# Validate configuration schema
cross-orch config lint --file deployment.yaml
# Check resource quotas
cross-orch quota check --cluster us-east-1a
```

**Step 2: Dry Run (optional)**
```bash
cross-orch deploy myapp --platform kubernetes --clusters eu-west,us-east --dry-run
# Output: kubectl apply --dry-run=server -f manifests/generated.yaml
```

**Step 3: Progressive Rollout**
```bash
# Phase 1: Deploy to staging cluster
cross-orch deploy myapp --platform kubernetes --clusters staging
# Phase 2: Smoke tests
cross-orch health myapp --cluster staging --deep
# Phase 3: Canary deployment (10% traffic)
cross-orch deploy myapp --platform kubernetes --clusters eu-west --traffic 10%
# Phase 4: Full rollout after AI approval
cross-orch deploy myapp --platform kubernetes --clusters eu-west --traffic 100%
```

**Step 4: Verification**
```bash
# Aggregate health checks across all clusters
cross-orch health myapp --all-clusters --deep
# Check metrics for anomalies
cross-orch metrics myapp --range 15m --percentiles p50,p95,p99
# Verify configuration sync
cross-orch config diff myapp
# Run smoke tests
cross-orch smoke-test myapp --endpoints http://myapp:8080/health
```

**Step 5: Telemetry & Learning**
```bash
# Record deployment outcome for AI model training
cross-orch telemetry record --deployment-id dep_abc123 --success true
# Update AI model with actual resource usage
cross-orch ai feedback --deployment-id dep_abc123 --metrics '{"cpu_avg": 65.2, "memory_peak": 2.3Gi}'
```

### Post-Execution Phase
1. **Success Path**: Update AI training data, notify stakeholders, document deployment
2. **Failure Path**: Trigger rollback automatically if health checks fail, create incident ticket
3. **Partial Success**: Record which clusters succeeded, pause before proceeding

## Golden Rules

1. **AI Recommendations Are Not Mandatory**: AI suggestions require human approval for:
   - Production cluster modifications
   - Cost overruns > 10% of budget
   - Service degradation scenarios
   - Cross-region failover

2. **Always Preserve Rollback Capability**: Every deployment must:
   - Store previous manifest in object storage (S3/GCS/Blob)
   - Maintain old replicas for 15 minutes post-deployment
   - Document rollback procedure in execution log
   - Test automation rollback weekly

3. **Multi-Cluster Consistency**:
   - Configuration must be identical across clusters unless explicitly tagged as cluster-specific
   - Secrets must use cluster-specific encryption keys
   - Health checks must pass in all clusters before marking deployment complete

4. **Quota Awareness**:
   - Check cluster resource quotas before scheduling
   - Validate that AI recommendations respect hard limits
   - Implement budget alerts at 80%, 90%, 100% of allocated spend

5. **Observability First**:
   - Every operation must log structured JSON to stdout
   - All changes should be annotated with run_id and operator_id
   - Metrics must be exported to Prometheus at `/metrics` endpoint
   - Maintain audit trail for 90 days (immutable storage)

6. **Platform Abstraction Leaks**:
   - Document any platform-specific workarounds in `platform-notes.md`
   - Flag deployments requiring platform-specific features
   - Provide escape hatches for direct platform CLI when needed

## Examples

### Example 1: AI-Optimized Multi-Cluster Deployment

**User Input:**
```bash
cross-orch deploy payment-service \
  --image registry.example.com/payment:v2.3 \
  --platform kubernetes \
  --clusters us-east-1a,us-east-1b,eu-west-1a \
  --replicas auto \
  --budget 5000/month \
  --optimize-for performance
```

**System Analysis:**
```
AI Recommendations (Confidence 92%):
  • us-east-1a: 4 replicas (latency 12ms, capacity: 70%)
  • us-east-1b: 3 replicas (latency 15ms, capacity: 85%)
  • eu-west-1a: 3 replicas (latency 45ms, capacity: 60%)

Expected Performance:
  • P50 latency: 48ms
  • P99 latency: 142ms
  • Throughput: 12,000 req/s
  • Estimated cost: $4,287/month

Risk Assessment: Low (historical success rate: 98%)
```

**System Execution:**
```
✓ Cluster connectivity verified
✓ Resource quotas confirmed
✓ Previous version v2.2 backed up to s3://crossorch-backups/dep_20240301_abc123
✓ Deploying to us-east-1a (4 replicas)
✓ Deploying to us-east-1b (3 replicas)
✓ Deploying to eu-west-1a (3 replicas)
✓ Health checks passed (100% success rate)
✓ Traffic shifted gradually:
   - Phase 1: 10% traffic to v2.3 (5 min)
   - Phase 2: 50% traffic (10 min)
   - Phase 3: 100% traffic (15 min)
✓ Configuration synchronized across clusters
✓ Metrics recording enabled

Deployment ID: dep_20240301_xyz789
Rollback ID: rollback_20240301_xyz789
```

**Verification:**
```bash
cross-orch health payment-service --all-clusters --deep
# Output:
# Cluster: us-east-1a
# ✅ All pods healthy (4/4)
# ✅ Endpoint http://10.0.1.10:8080/health returns 200
# ✅ Database connectivity: OK
# Cluster: us-east-1b
# ✅ All pods healthy (3/3)
# ✅ Endpoint http://10.0.2.10:8080/health returns 200
# Cluster: eu-west-1a
# ✅ All pods healthy (3/3)
# ✅ Endpoint http://10.0.3.10:8080/health returns 200
# Overall Status: HEALTHY (10/10 pods)
```

### Example 2: Cross-Platform Database Migration

**User Input:**
```bash
cross-orch migrate database \
  --source docker-swarm postgres-primary \
  --target kubernetes postgres-cluster \
  --mode replication-sync \
  --cutover-window 02:00-04:00 UTC \
  --verify-checksum
```

**System Execution Process:**
```
Phase 1: Pre-migration validation (--dry-run)
  ✓ Source database accessible (PostgreSQL 14.5)
  ✓ Target cluster has capacity (16 vCPU, 64Gi RAM)
  ✓ Network connectivity: 5ms latency, 10Gbps
  ✓ Storage: 2Ti available on target

Phase 2: Initial data sync
  Starting logical replication...
  Progress: ████████████████████ 100% (2.1Ti / 2.1Ti)
  WAL position: 0/20000000
  Lag: 0 bytes
  Duration: 2h 14m

Phase 3: Catch-up replication
  Monitoring replication lag:
    2024-03-01T02:15:00Z: lag=0MB (healthy)
    2024-03-01T02:45:00Z: lag=0MB (healthy)
    2024-03-01T03:15:00Z: lag=0MB (healthy)

Phase 4: Application reconfiguration
  ✓ Updating connection strings in config maps:
    • us-east-1a: DB_HOST=postgres-cluster.production.svc
    • us-east-1b: DB_HOST=postgres-cluster.production.svc
    • eu-west-1a: DB_HOST=postgres-cluster.production.svc
  ✓ Rolling restart of application pods (0 downtime)
  ✓ Verifying application connectivity:
    • payment-api: connected (query latency 2ms)
    • user-service: connected (query latency 3ms)
    • analytics: connected (query latency 5ms)

Phase 5: Cutover completion
  ✓ Source database set to read-only
  ✓ Final sync delta applied (42 MB, 3 seconds)
  ✓ Monitoring alerts updated
  ✓ Documentation updated
  ✓ Backup of source retained for 30 days

Migration completed successfully!
Cutover window: 04:17-04:20 UTC (3 minutes)
Data loss: 0 rows
Application downtime: 0 seconds
```

**Rollback Command (if needed):**
```bash
cross-orch migrate rollback \
  --migration-id mig_20240301_abc123 \
  --target-source \
  --verify-consistency

# This would:
# 1. Switch application back to source database
# 2. Stop replication from target
# 3. Validate all write operations return to source
# 4. Clean up target cluster resources
```

### Example 3: Emergency Failover with AI Analysis

**Scenario:** Database cluster in us-east-1a experiencing 10% packet loss

**User Input:**
```bash
cross-orch cluster failover \
  --service payment-database \
  --from us-east-1a \
  --to us-east-1b \
  --verify-data-consistency \
  --maintenance-mode
```

**System Response:**
```
AI Analysis of failover impact:
  • Current health: DEGRADED (10% packet loss detected)
  • Estimated failover time: 45-90 seconds
  • Data loss risk: < 0.001% (synchronous replication enabled)
  • Performance impact: 5% latency increase expected
  • Cost impact: $0.12/hour additional (overlap during cutover)

Executing controlled failover:
─────────────────────────────────────────────────────
Step 1/5: Pre-failover snapshot (30s)
  ✓ Database backup created
  ✓ WAL position recorded: 0/30000000
  ✓ Replication lag: 0ms

Step 2/5: Promote standby cluster (15s)
  ✓ us-east-1b: Promoting to primary
  ✓ Streaming replication active
  ✓ WAL receiver started

Step 3/5: Application reconfiguration (45s)
  Updating connection pools (rolling restart):
    • payment-api: 4/5 pods updated
    • user-service: 6/6 pods updated
    • analytics: 3/3 pods updated
  ✓ All services pointing to us-east-1b
  ✓ Connection pool warm-up (30s)

Step 4/5: Traffic validation (60s)
  ✓ Health checks passing (100%)
  ✓ Database queries: 2ms avg latency
  ✓ Replication lag: 0ms
  ✓ Monitoring alerts: NORMAL

Step 5/5: Post-failover cleanup (30s)
  ✓ Old primary set to hot standby
  ✓ Alerts updated for new primary
  ✓ Documentation updated
  ✓ Runbook triggered: FAILOVER_COMPLETED

─────────────────────────────────────────────────────
Failover completed in 3m 12s
Data consistency: VERIFIED (checksum match)
Applications impact: ZERO DOWNTIME
New primary: us-east-1b
Rollback window: 15 minutes (automatic quiesce)
```

**Rollback Command (if needed within 15 min window):**
```bash
cross-orch cluster rollback-failover \
  --migration-id failover_20240301_def456 \
  --to us-east-1a \
  --force

# Immediate return to original primary
# Requires: --force if 15 minute window expired
```

### Example 4: Predictive Autoscaling

**User Input:**
```bash
cross-orch autoscale web-app \
  --min-replicas 3 \
  --max-replicas 50 \
  --ai-optimization \
  --predict-window 7d \
  --budget 2500/month \
  --sla p95-latency<100ms
```

**AI Analysis and Auto-Configuration:**
```
Historical Analysis (90 days):
  • Peak load patterns:
    - Weekdays 09:00-11:00 UTC: 8,000 req/s (avg)
    - Weekdays 14:00-16:00 UTC: 6,500 req/s (avg)
    - Weekends: 1,200 req/s (avg)
  • Current resource usage:
    - CPU: 65% average, 89% peak
    - Memory: 2.8Gi per pod (of 4Gi limit)
    - Network: 45Mbps per pod
  
AI Recommendations:
  • Baseline: 5 replicas (covers 80% of traffic)
  • Scaling triggers:
    - Scale up: CPU > 70% OR Latency > 80ms (2 consecutive periods)
    - Scale down: CPU < 30% AND Latency < 50ms (5 consecutive periods)
  • Maximum burst: 15 replicas (handles 99.9% of historical peaks)
  • Cost optimization:
    - Use spot instances for 60% of replicas (savings: 65%)
    - Reserved instances for baseline (3 replicas)
  
Predicted Monthly Cost: $1,847 (vs static 20 replicas: $3,200)
Predicted SLA compliance: 99.92% (vs target: 99.9%)
```

**Real-time Scaling Events (sample log):**
```
2024-03-01T09:02:15Z: Scaling event triggered
  Reason: CPU utilization 72% (threshold: 70%)
  Current replicas: 5 → 7
  AI confidence: 94%
  Cost impact: +$2.34/hour

2024-03-01T09:15:42Z: Scaling event triggered
  Reason: Latency P95 = 85ms (threshold: 80ms)
  Current replicas: 7 → 10
  AI confidence: 91%
  Cost impact: +$3.87/hour

2024-03-01T11:30:00Z: Scale down event
  Reason: CPU 28% AND Latency 45ms (consecutive 5 periods)
  Current replicas: 10 → 8
  AI confidence: 88%
  Cost savings: -$1.55/hour
```

**Verification:**
```bash
cross-orch autoscale status web-app
# Output:
# Mode: AI-driven (model: orchestrator-v3.1)
# Current replicas: 8
# Min/Max: 3/50
# Scaling history (24h):
#   • 2 scale-up events, 3 scale-down events
#   • Total cost: $184.20 (vs baseline $312.00)
#   • SLA compliance: 99.95%
#   • AI decision accuracy: 94.2%
```

## Rollback Commands

### 1. Deployment Rollback

**Command:** `cross-orch rollback <deployment-id>`

**Options:**
- `--to <version|revision>`: Target specific version or revision number
- `--clusters <list>`: Rollback only specific clusters (default: all)
- `--force`: Skip health checks
- `--timeout <duration>`: Max wait time (default: 300s)
- `--keep-replicas`: Maintain current replica count

**Example:**
```bash
cross-orch rollback dep_20240301_xyz789 \
  --to v2.2 \
  --clusters eu-west-1a,us-east-1b \
  --timeout 600s
```

**What it does:**
1. Retrieves manifest from backup storage (S3/GCS)
2. Validates previous version is compatible
3. Executes rolling back to old replicas:
   - Phase 1: Start old replicas (gradually while old new replicas still running)
   - Phase 2: Wait for old pods to become ready
   - Phase 3: Scale down new replicas gradually
   - Phase 4: Update services to point to old pods
4. Verifies health checks return to baseline
5. Updates deployment record with rollback status

**Rollback ID:** Generated as `rollback_<deployment-id>_<timestamp>`

**Rollback Verification:**
```bash
cross-orch rollback status <rollback-id>
# Output:
# Rollback Status: COMPLETED
# Source deployment: dep_20240301_xyz789
# Target version: v2.2
# Clusters: eu-west-1a (5 pods), us-east-1b (3 pods)
# Duration: 4m 32s
# Health: ✅ All pods healthy
# Traffic: 100% to old version
```

### 2. Configuration Rollback

**Command:** `cross-orch config rollback <config-revision>`

**Options:**
- `--clusters <list>`: Specific clusters
- `--dry-run`: Preview changes
- `--validate`: Run schema validation before apply
- `--timeout <duration>`: Max wait for propogation

**Example:**
```bash
cross-orch config rollback config_rev_20240301_abc123 \
  --clusters all \
  --validate
```

**Internal Mechanics:**
1. Retrieve previous config from revision history (etcd/Consul)
2. Compute diff between current and target config
3. Validate target config against schema
4. Apply using platform-native commands:
   - Kubernetes: `kubectl apply -f <config> --server-side`
   - Docker: `docker service update --force <service>`
   - Nomad: `nomad job plan <job.hcl> && nomad job run`
5. Wait for propogation (10s default)
6. Verify no config drift

### 3. Cluster Failover Rollback

**Command:** `cross-orch cluster rollback-failover <migration-id>`

**Options:**
- `--to <cluster-name>`: Target cluster (default: original)
- `--force`: Bypass health checks and data consistency validation
- `--timeout <duration>`: Max wait time
- `--skip-data-validation`: For non-critical services

**Example:**
```bash
cross-orch cluster rollback-failover failover_20240301_def456 \
  --to us-east-1a \
  --timeout 300s
```

**Rollback Process:**
1. Check if original cluster is healthy (skip with --force)
2. Promote original cluster back to primary (if demoted)
3. Reconfigure application to point back
4. Validate data consistency (checksum replication logs)
5. Demote secondary back to standby
6. Update monitoring and alerts

**Important:** Failover rollback within 15 minutes is typically seamless. Beyond that requires careful data sync verification.

### 4. AI Decision Rollback

**Command:** `cross-orch ai undo <decision-id>`

**Options:**
- `--scope <deployment|config|cluster>`: Type of decision
- `--dry-run`: Show what would be undone
- `--partial`: Undo only specific affected resources

**Example:**
```bash
cross-orch ai undo decision_20240301_xyz789 \
  --scope deployment \
  --dry-run
```

**What it reverts:**
- All deployment changes made based on AI recommendation
- Configuration modifications
- Autoscaling policy changes
- Resource quota adjustments
- Cost allocation tags

### 5. Infrastructure Rollback

**Command:** `cross-orch infra rollback <resource-id>`

**Options:**
- `--snapshot-id`: Rollback to specific snapshot
- `--latest`: Rollback to most recent healthy state
- `--force`: Destroy current resources even if healthy
- `--dry-run`: Plan the rollback

**Example:**
```bash
cross-orch infra rollback vm_20240301_abc123 \
  --latest \
  --dry-run
```

**Process:**
1. Identify current state of resource
2. Find appropriate backup/snapshot
3. Validate rollback won't cause data loss
4. Create new resource from snapshot (if immutable)
5. Switch DNS/load balancer to new resource
6. Destroy old resource (optional with --keep-old)
7. Update resource registry

### Rollback Best Practices

1. **Test rollback before deployment**: Always dry-run your rollback command
2. **Maintain backup windows**: Ensure backups exist before operations
3. **Document rollback procedures**: Include rollback commands in runbooks
4. **Time box rollbacks**: Set `--timeout` appropriate to service criticality
5. **Partial rollbacks**: Roll back only affected clusters if possible
6. **Post-rollback validation**: Always verify service health after rollback
7. **Learn from failures**: Feed rollback events into AI model to improve recommendations

## Dependencies

### Required System Packages
- `kubectl` >= 1.28 (Kubernetes CLI)
- `docker` >= 24.0 (Docker CLI with Swarm)
- `nomad` >= 1.6.0 (HashiCorp Nomad)
- `python3` >= 3.10 with packages:
  - `pyyaml>=6.0`
  - `requests>=2.31`
  - `kubernetes>=28.0.0`
  - `docker>=6.1.0`
  - `python-nomad>=2.2.0`
  - `tensorflow-serving-api>=2.12`
  - `prometheus-client>=0.18`

### Optional Dependencies
- `terraform` >= 1.5 (for infra provisioning)
- `helm` >= 3.12 (for Helm chart deployments)
- `awscli` >= 2.0 (for AWS integrations)
- `gcloud` >= 450.0.0 (for GCP integrations)
- `az` >= 2.50 (for Azure integrations)

### AI/ML Components
- TensorFlow Serving (for model inference)
- Cross Orchestrator AI model (orchestrator-v3.1+)
- Model storage: S3/GCS/Blob with path `models/orchestrator/`
- Model training data pipeline (separate service)

### Minimum System Requirements
- CPU: 4+ cores
- Memory: 8GiB minimum, 16GiB recommended
- Storage: 20GiB for models and logs
- Network: Outbound HTTPS to cluster APIs

## Environment Variables

### Core Configuration
```bash
# Cluster credentials
export CROSS_ORCH_KUBECONFIG="$HOME/.kube/config"
export CROSS_ORCH_DOCKER_HOST="unix:///var/run/docker.sock"
export CROSS_ORCH_NOMAD_ADDR="http://[10.0.0.1]:4646"
export CROSS_ORCH_SWARM_ADDR="tcp://swarm-manager:2377"

# AI model configuration
export CROSS_ORCH_AI_MODEL_PATH="/opt/cross-orch/models/orchestrator-v3.1"
export CROSS_ORCH_AI_MODEL_URL="http://localhost:8501/v1/models/orchestrator"
export CROSS_ORCH_AI_CONFIDENCE_THRESHOLD="0.75"
export CROSS_ORCH_AI_ENABLED="true"

# Backup and state storage
export CROSS_ORCH_BACKUP_BUCKET="crossorch-backups"
export CROSS_ORCH_BACKUP_REGION="us-east-1"
export CROSS_ORCH_STATE_STORE="consul://consul.crossorch.io:8500"
export CROSS_ORCH_STATE_KEYSPACE="cross-orch/v1"

# Telemetry and logging
export CROSS_ORCH_TELEMETRY_ENDPOINT="https://telemetry.crossorch.io/v1/metrics"
export CROSS_ORCH_LOG_LEVEL="INFO"
export CROSS_ORCH_JSON_LOGGING="true"
export CROSS_ORCH_AUDIT_LOG="/var/log/cross-orch/audit.jsonl"
export CROSS_ORCH_METRICS_PORT="9090"

# Cost and budget management
export CROSS_ORCH_COST_BUDGET="5000"
export CROSS_ORCH_COST_CURRENCY="USD"
export CROSS_ORCH_COST_ALERT_THRESHOLD="0.80"
export CROSS_ORCH_COST_TRACKING_ENABLED="true"

# Multi-cluster configuration
export CROSS_ORCH_DEFAULT_CLUSTERS="us-east-1a,us-east-1b,eu-west-1a"
export CROSS_ORCH_CLUSTER_REGISTRY="config://clusters.yaml"
export CROSS_ORCH_CLUSTER_TIMEOUT="30s"
export CROSS_ORCH_CLUSTER_MAX_RETRIES="3"

# Feature flags
export CROSS_ORCH_ENABLE_AI="true"
export CROSS_ORCH_ENABLE_AUTO_ROLLBACK="true"
export CROSS_ORCH_ENABLE_CANARY="true"
export CROSS_ORCH_ENABLE_MULTI_REGION="true"
export CROSS_ORCH_ENABLE_COST_OPTIMIZATION="true"

# Integration hooks (webhooks)
export CROSS_ORCH_WEBHOOK_URL="https://hooks.slack.com/services/..."
export CROSS_ORCH_WEBHOOK_ON_SUCCESS="true"
export CROSS_ORCH_WEBHOOK_ON_FAILURE="true"
export CROSS_ORCH_WEBHOOK_ON_ROLLBACK="true"

# Security
export CROSS_ORCH_VAULT_ADDR="https://vault.crossorch.io"
export CROSS_ORCH_VAULT_PATH="secret/cross-orch"
export CROSS_ORCH_TLS_CERT="/etc/cross-orch/tls/cert.pem"
export CROSS_ORCH_TLS_KEY="/etc/cross-orch/tls/key.pem"
```

### Configuration Files
- `~/.config/cross-orch/config.yaml`: Main configuration
- `~/.config/cross-orch/clusters.yaml`: Cluster definitions
- `/etc/cross-orch/models/orchestrator-v3.1/`: AI model directory
- `/var/lib/cross-orch/state/`: Persistent state storage

## Verification

After executing any operation, verify correctness:

### 1. Deployment Verification
```bash
# Check pod status across all clusters
for cluster in $(cross-orch cluster list --quiet); do
  echo "=== Cluster: $cluster ==="
  kubectl --context $cluster get pods -l app=myapp -o wide
done

# Verify replica count matches expectation
cross-orch scale status myapp --clusters all

# Test application endpoints
cross-orch health myapp --all-clusters --deep

# Check logs for errors
cross-orch logs myapp --tail 100 --since 10m --clusters all

# Verify service discovery
cross-orch endpoints myapp --clusters all
```

### 2. Configuration Verification
```bash
# Compare configs across clusters
cross-orch config diff myapp

# Validate config schema
cross-orch config lint --file deployed-config.yaml

# Check config propogation latency
cross-orch config status myapp --detail

# Verify secrets are present
cross-orch secrets list --cluster us-east-1a | grep myapp
```

### 3. AI Optimization Verification
```bash
# Check AI recommendation confidence
cross-orch ai recommendations myapp --limit 5

# Review AI decision logs
cross-orch ai audit log --deployment-id dep_abc123

# Validate predictions against reality
cross-orch ai evaluate myapp --range 24h

# Check model version
cross-orch ai model info
```

### 4. Cost Verification
```bash
# Show actual vs predicted cost
cross-orch cost report --deployment-id dep_abc123

# Compare cost across clusters
cross-orch cost breakdown --cluster us-east-1a,us-east-1b,eu-west-1a

# Verify budget compliance
cross-orch cost budget status

# Identify cost anomalies
cross-orch cost anomalies --threshold 20%
```

### 5. Failover Verification
```bash
# Verify failover completed
cross-orch cluster status --detail

# Check replication lag (if databases)
cross-orch db replication lag --service payment-db

# Validate monitoring alerts
cross-orch alerts list --severity critical

# Test application connectivity to new primary
curl -s http://payment-api.production.svc:8080/health
```

### 6. Integration Verification
```bash
# Check webhook delivery status
cross-orch webhook log --since 1h

# Verify CI/CD pipeline integration
cross-orch pipeline status --deployment-id dep_abc123

# Check audit trail
cross-orch audit log --deployment-id dep_abc123 --format json

# Validate RBAC permissions
cross-orch auth whoami --cluster all
```

## Troubleshooting

### Issue 1: `ERROR: Unable to connect to cluster <name>`

**Symptoms:**
```bash
$ cross-orch cluster status
ERROR: Unable to connect to cluster us-east-1a: connection refused
```

**Possible Causes & Solutions:**

1. **kubeconfig incorrect**
   ```bash
   # Verify kubeconfig exists
   ls -la "$CROSS_ORCH_KUBECONFIG"
   
   # Test kubectl connectivity
   kubectl --context us-east-1a get nodes
   
   # Update if needed
   cross-orch cluster update us-east-1a --kubeconfig /path/to/new/config
   ```

2. **Network/firewall blocking**
   ```bash
   # Test network connectivity
   nc -zv api.us-east-1a.eks.amazonaws.com 443
   
   # Check AWS security groups allow your IP
   aws ec2 describe-security-groups --group-ids sg-xxxx
   
   # If VPN needed, establish connection
   openvpn --config client.ovpn
   ```

3. **Expired credentials**
   ```bash
   # For AWS EKS, check token expiration
   kubectl --context us-east-1a get pods
   
   # If using aws-iam-authenticator, refresh credentials
   aws eks update-kubeconfig --name cluster-name --region us-east-1a
   
   # For GKE, regenerate gcloud credentials
   gcloud container clusters get-credentials cluster-name --region us-east-1a
   ```

4. **Cluster API endpoint changed**
   ```bash
   # Update cluster endpoint
   cross-orch cluster update us-east-1a --endpoint new-api.endpoint.com
   
   # Re-run cluster validation
   cross-orch cluster validate us-east-1a
   ```

**Diagnostic Command:**
```bash
cross-orch debug connectivity \
  --cluster us-east-1a \
  --checks dns,tls,auth,pods \
  --verbose
```

### Issue 2: `AI Model Inference Failed: Model not found`

**Symptoms:**
```bash
$ cross-orch autoscale web-app --ai-optimization
ERROR: AI model inference failed: Model orchestrator-v3.1 not found at /opt/cross-orch/models/orchestrator-v3.1
```

**Possible Causes & Solutions:**

1. **Model not installed**
   ```bash
   # Check if model directory exists
   ls -la "$CROSS_ORCH_AI_MODEL_PATH"
   
   # If empty, download model
   mkdir -p "$CROSS_ORCH_AI_MODEL_PATH"
   aws s3 sync s3://crossorch-models/orchestrator-v3.1/ "$CROSS_ORCH_AI_MODEL_PATH/"
   
   # Verify model files
   ls -la "$CROSS_ORCH_AI_MODEL_PATH"/
   # Should see: saved_model.pb, variables/, assets/
   ```

2. **TensorFlow Serving not running**
   ```bash
   # Check TensorFlow Serving service
   systemctl status tensorflow-serving
   
   # Start if stopped
   sudo systemctl start tensorflow-serving
   sudo systemctl enable tensorflow-serving
   
   # Verify model loaded via API
   curl -s http://localhost:8501/v1/models/orchestrator | jq .
   # Should show: {"model_version_status":[{"version":"3.1","state":"AVAILABLE"}]}
   ```

3. **Model version mismatch**
   ```bash
   # Check expected version in config
   cross-orch config get ai.model_version
   
   # Update env var to match installed version
   export CROSS_ORCH_AI_MODEL_PATH="/opt/cross-orch/models/orchestrator-v3.2"
   
   # Or download correct version
   aws s3 sync s3://crossorch-models/orchestrator-v3.2/ "$CROSS_ORCH_AI_MODEL_PATH/"
   ```

4. **Insufficient memory for model loading**
   ```bash
   # Check system memory
   free -h
   
   # Increase TensorFlow Serving memory limit
   # Edit /etc/systemd/system/tensorflow-serving.service
   # Add: --model_config_file_poll_wait_seconds=60
   # Restart: sudo systemctl restart tensorflow-serving
   
   # Reduce batch size for inference
   export CROSS_ORCH_AI_BATCH_SIZE="16"
   ```

**Diagnostic Command:**
```bash
cross-orch debug ai-model \
  --model-path "$CROSS_ORCH_AI_MODEL_PATH" \
  --test-input '{"cpu_util": 75.2, "memory_util": 68.5, "latency_p95": 120}' \
  --verbose
```

### Issue 3: `Configuration drift detected between clusters`

**Symptoms:**
```bash
$ cross-orch config diff myapp
--- eu-west-1a
+++ us-east-1a
@@ -1,12 +1,12 @@
 replicas: 5
-memory_limit: 4Gi
+memory_limit: 8Gi
 cpu_limit: 2000m
```

**Possible Causes & Solutions:**

1. **Manual changes bypassed Cross Orchestrator**
   ```bash
   # Identify who made manual change
   kubectl --context eu-west-1a get deployment myapp -o yaml | grep -A2 "annotations:"
   
   # Re-sync from source of truth
   cross-orch config push myapp \
     --source config-repository \
     --clusters eu-west-1a \
     --force
   ```

2. **Config sync failure**
   ```bash
   # Check sync status
   cross-orch config status myapp --detail
   
   # View sync logs
   journalctl -u cross-orch-config-sync -n 100
   
   # Retry sync
   cross-orch config sync myapp --clusters all --force
   ```

3. **Cluster-specific overrides**
   ```bash
   # Check for cluster-specific labels
   kubectl --context eu-west-1a get deployment myapp -o yaml | grep -A5 "nodeSelector"
   
   # If intentional, document and approve drift
   cross-orch config annotate myapp \
     --cluster eu-west-1a \
     --key drift.allowed \
     --value "true" \
     --reason "GPU nodes required in eu-west"
   ```

4. **Concurrent configuration changes**
   ```bash
   # Identify conflicting changes
   cross-orch config conflicts myapp
   
   # Resolve using latest approved version
   cross-orch config resolve myapp \
     --strategy latest-approved \
     --notify-owner
   ```

**Prevention:**
```bash
# Enable config drift detection alerts
cross-orch alerts create \
  --name "Config Drift" \
  --condition "config_drift > 0" \
  --severity warning \
  --notify-on slack:#infrastructure

# Set config sync schedule
cross-orch config schedule \
  --interval 5m \
  --auto-fix \
  --audit
```

### Issue 4: `Rollback timed out waiting for pods to become ready`

**Symptoms:**
```bash
$ cross-orch rollback dep_20240301_xyz789
ERROR: Rollback timed out after 300s waiting for pods to become ready
  • eu-west-1a: 5 pods stuck in "ContainerCreating" or "CrashLoopBackOff"
```

**Possible Causes & Solutions:**

1. **Rollback image not available**
   ```bash
   # Check if image exists in registry
   docker manifest inspect registry.example.com/app:v2.2
   
   # Pull image manually to verify
   docker pull registry.example.com/app:v2.2
   
   # Update image pull secrets if needed
   cross-orch secrets update registry-creds \
     --username "newuser" \
     --password "newpass"
   ```

2. **Insufficient resources on nodes**
   ```bash
   # Check node capacity
   kubectl --context eu-west-1a describe node <node-name> | grep -A5 "Allocated resources"
   
   # Scale down current deployment to free resources
   cross-orch scale myapp --replicas 1 --clusters eu-west-1a
   
   # Add node pool (infra command)
   cross-orch infra create node-pool \
     --cluster eu-west-1a \
     --count 2 \
     --instance-type m5.xlarge
   ```

3. **Application startup failure**
   ```bash
   # Check pod logs
   kubectl --context eu-west-1a logs --previous <pod-name> --tail 100
   
   # Check events
   kubectl --context eu-west-1a describe pod <pod-name> | grep -A20 "Events:"
   
   # Common fixes:
   # - Set environment variables
   cross-orch config set myapp DB_HOST=postgres.production.svc --cluster eu-west-1a
   # - Increase resources
   cross-orch config set myapp resources.memory="8Gi" --cluster eu-west-1a
   ```

4. **Rollback hitting health check timeout**
   ```bash
   # Extend health check timeout
   cross-orch rollback dep_20240301_xyz789 \
     --health-timeout 600s \
     --force
   
   # Or skip health checks (use with caution)
   cross-orch rollback dep_20240301_xyz789 \
     --no-health-check \
     --clusters eu-west-1a
   ```

5. **Node taints/tolerations mismatch**
   ```bash
   # Check node taints
   kubectl --context eu-west-1a describe node <node-name> | grep Taint
   
   # Add tolerations to deployment
   cross-orch config patch myapp \
     --cluster eu-west-1a \
     --patch '{"spec":{"template":{"spec":{"tolerations":[{"key":"dedicated","operator":"Equal","value":"production","effect":"NoSchedule"}]}}}}'
   ```

**Emergency Measures:**
```bash
# 1. Partial rollback (only healthy clusters)
cross-orch rollback dep_20240301_xyz789 \
  --clusters us-east-1a,us-east-1b \
  --exclude eu-west-1a

# 2. Manual kubectl rollback
kubectl --context eu-west-1a rollout undo deployment/myapp

# 3. Force delete stuck pods (last resort)
kubectl --context eu-west-1a delete pod <pod-name> --force --grace-period=0

# 4. After manual intervention, resume automation
cross-orch rollback resume dep_20240301_xyz789
```

### Issue 5: `Cost budget exceeded: Predicted $5,287 > Budget $5,000`

**Symptoms:**
```bash
$ cross-orch deploy myapp --budget 5000
ERROR: Cost budget exceeded
  Predicted cost: $5,287/month
  Budget: $5,000/month
  Overrun: 5.7%
```

**Solutions:**

1. **Reduce replica count**
   ```bash
   cross-orch deploy myapp \
     --replicas 4 \
     --budget 5000 \
     --optimize-for cost
   ```

2. **Use spot instances**
   ```bash
   cross-orch deploy myapp \
     --platform kubernetes \
     --node-selector "cloud.google.com/gke-preemptible=true" \
     --spot-percent 80
   ```

3. **Reduce resource requests**
   ```bash
   # Analyze actual usage
   cross-orch metrics myapp --range 7d --percentiles p50,p95
   
   # Adjust resources based on p95
   cross-orch config set myapp \
     resources.cpu="500m" \
     resources.memory="2Gi"
   ```

4. **Select cheaper regions**
   ```bash
   cross-orch deploy myapp \
     --clusters us-west-2a,us-west-2b,us-central-1a \
     --budget 5000
   ```

5. **Use reserved instances/savings plans**
   ```bash
   # Configure cloud provider reservations
   cross-orch infra create reservation \
     --instance-type m5.large \
     --term 1-year \
     --payment-option partial-upfront
   
   # Or let AI optimize
   cross-orch ai optimize myapp --objective cost --budget 5000
   ```

**Budget Alert Configuration:**
```bash
# Set up proactive alerts at 80% budget usage
cross-orch budget create \
  --name "Production Apps" \
  --budget 5000 \
  --alert-at 80,90,100 \
  --notify slack:#infrastructure-alerts

# Enable cost exploration mode (suggests optimizations)
export CROSS_ORCH_COST_EXPLORATION_MODE="true"
```

### Issue 6: `AI recommendation confidence too low: 42%`

**Symptoms:**
```bash
$ cross-orch autoscale myapp --ai-optimization
WARNING: AI recommendation confidence 42% below threshold 75%
Recommendation: Use conservative scaling rules instead
```

**Solutions:**

1. **Wait for more training data**
   ```bash
   # Check how much training data exists
   cross-orch ai training-data-count --service myapp
   
   # If < 1000 data points, AI may not be reliable yet
   # Use rule-based scaling until sufficient data collected
   cross-orch autoscale myapp --ai-optimization=false
   ```

2. **Improve data quality**
   ```bash
   # Check data freshness
   cross-orch ai data-stats --service myapp
   
   # Ensure metrics are being collected:
   # - CPU, memory, network, disk
   # - Request rates, error rates, latencies
   # - Custom business metrics
   
   # Enable additional metrics collection
   cross-orch metrics enable \
     --service myapp \
     --metrics request_rate,error_rate,p99_latency
   ```

3. **Adjust confidence threshold**
   ```bash
   export CROSS_ORCH_AI_CONFIDENCE_THRESHOLD="0.60"
   
   # Or per-command override
   cross-orch autoscale myapp \
     --ai-optimization \
     --min-confidence 0.60
   ```

4. **Use AI in advisory mode**
   ```bash
   # AI provides recommendations but doesn't auto-apply
   cross-orch autoscale myapp --ai-optimization --require-approval
   
   # Review recommendations
   cross-orch ai recommendations myapp --format json
   
   # Apply manually if you agree
   cross-orch scale myapp 12 --reason "AI recommendation: expected demand spike"
   ```

5. **Retrain model with more data**
   ```bash
   # Trigger model retraining
   cross-orch ai retrain \
     --include-services myapp,payment-service \
     --since 30d \
     --wait
   
   # Check training status
   cross-orch ai training-status
   ```

### Issue 7: `Multi-cluster deployment incomplete: 2/5 clusters failed`

**Symptoms:**
```bash
$ cross-orch deploy myapp --clusters us-east-1a,us-east-1b,eu-west-1a,ap-southeast-1a,ap-southeast-1b
✅ us-east-1a: Deployed (5 replicas)
✅ us-east-1b: Deployed (5 replicas)
⚠️ eu-west-1a: Health check failing (see warnings)
❌ ap-southeast-1a: Quota exceeded
❌ ap-southeast-1b: Connection timeout
```

**Troubleshooting Steps:**

1. **Check which clusters failed and why**
   ```bash
   cross-orch deployment status <deployment-id> --detail
   
   # Shows per-cluster status with error messages
   ```

2. **Fix quota issues**
   ```bash
   # Check current quota
   kubectl --context ap-southeast-1a describe quota
   
   # Request quota increase via cloud provider
   aws service-quotas request-service-quota-increase \
     --service-code ec2 \
     --quota-code L-XXXXXX \
     --desired-value 100
   
   # Or use smaller footprint
   cross-orch config set myapp \
     resources.cpu="250m" \
     resources.memory="1Gi" \
     --cluster ap-southeast-1a
   ```

3. **Fix connectivity issues**
   ```bash
   # Test connectivity to failing cluster
   cross-orch cluster ping ap-southeast-1b
   
   # If network issue, check:
   # - VPN/routing
   # - Firewall rules
   # - Cluster API endpoint availability
   
   # Retry failed clusters only
   cross-orch deploy myapp \
     --deployment-id <deployment-id> \
     --clusters ap-southeast-1a,ap-southeast-1b \
     --retry
   ```

4. **Fix health check issues**
   ```bash
   # Investigate why eu-west-1a health check failing
   cross-orch health myapp --cluster eu-west-1a --deep
   
   # Common causes:
   # - Missing configmaps/secrets
   cross-orch config sync myapp --clusters eu-west-1a --force
   
   # - Database connectivity issues
   cross-orch db connectivity-test --service myapp --cluster eu-west-1a
   
   # - Insufficient resources
   kubectl --context eu-west-1a top pods -l app=myapp
   ```

5. **Partial success handling**
   ```bash
   # Mark deployment as partially successful
   cross-orch deployment update <deployment-id> \
     --status partially-successful \
     --note "2/5 clusters successful, retrying failed clusters"
   
   # Continue retry logic for failed clusters
   cross-orch deployment retry <deployment-id> \
     --clusters ap-southeast-1a,ap-southeast-1a \
     --max-attempts 3
   ```

6. **Quarantine problematic clusters**
   ```bash
   # Temporarily exclude problematic clusters from auto-scaling
   cross-orch cluster annotate ap-southeast-1a \
     --key cross-orch.ignore \
     --value "true" \
     --reason "quota issues, maintenance in progress"
   
   # Re-include once fixed
   cross-orch cluster annotate ap-southeast-1a \
     --key cross-orch.ignore \
     --value "false"
   ```

**Prevention:**
```bash
# Pre-flight checks before deployment
cross-orch preflight deploy myapp \
  --clusters ap-southeast-1a,ap-southeast-1b \
  --checks quota,connectivity,compatibility

# Set cluster priority tiers
cross-orch cluster tier set \
  --cluster us-east-1a \
  --tier critical

cross-orch cluster tier set \
  --cluster ap-southeast-1b \
  --tier best-effort
```

### Issue 8: `Cross-platform deployment failed: Unsupported platform combination`

**Symptoms:**
```bash
$ cross-orch deploy myapp --platform kubernetes,docker --clusters k8s-cluster,swarm-cluster
ERROR: Unsupported platform combination: Cannot deploy to both Kubernetes and Docker Swarm in same deployment
```

**Solution:**
1. **Use separate deployments for each platform**
   ```bash
   cross-orch deploy myapp \
     --platform kubernetes \
     --clusters k8s-cluster \
     --image registry.example.com/myapp:latest
   
   cross-orch deploy myapp \
     --platform docker \
     --clusters swarm-cluster \
     --image registry.example.com/myapp:latest \
     --network overlay-network
   ```

2. **Platform-agnostic deployment (if supported)**
   ```bash
   # Some services can be deployed to any platform
   cross-orch deploy myapp \
     --clusters k8s-cluster,swarm-cluster \
     --platform auto-detect \
     --adapt-platform-config
   ```

3. **Use different service names for platform distinction**
   ```bash
   cross-orch deploy myapp-k8s \
     --platform kubernetes \
     --clusters k8s-cluster
   
   cross-orch deploy myapp-swarm \
     --platform docker \
     --clusters swarm-cluster
   ```

**Documentation Note:** Cross-platform deployments in a single command are not supported due to fundamental differences in service definitions. Use separate deployments with shared configuration base.

### Issue 9: `Service discovery failed: DNS resolution error`

**Symptoms:**
```bash
$ cross-orch endpoints myapp
ERROR: Service myapp not resolvable in cluster us-east-1a
  NXDOMAIN from 10.0.0.10:53
```

**Troubleshooting:**

1. **Check service exists**
   ```bash
   kubectl --context us-east-1a get service myapp
   # Should return service definition
   ```

2. **Check DNS configuration**
   ```bash
   kubectl --context us-east-1a get pods -l app=myapp -o jsonpath='{.items[0].status.podIP}'
   
   # Test DNS resolution from within cluster
   kubectl --context us-east-1a run dns-test --image=busybox --rm -it -- nslookup myapp
   
   # Check CoreDNS/kube-dns pods
   kubectl --context us-east-1a get pods -n kube-system -l k8s-app=kube-dns
   ```

3. **Verify service selectors match pod labels**
   ```bash
   kubectl --context us-east-1a get service myapp -o yaml | grep selector
   kubectl --context us-east-1a get pods -l app=myapp --show-labels
   
   # If mismatched, fix labels or service selector
   cross-orch config patch myapp \
     --cluster us-east-1a \
     --patch '{"spec":{"selector":{"app":"myapp"}}}'
   ```

4. **Check network policies**
   ```bash
   kubectl --context us-east-1a get networkpolicy -l app=myapp
   # If restrictive, allow DNS traffic
   ```

**Resolution Commands:**
```bash
# Recreate service with correct selector
cross-orch config delete-service myapp --cluster us-east-1a
cross-orch config create-service myapp --cluster us-east-1a --port 8080

# Flush DNS cache (if using custom DNS)
kubectl --context us-east-1a exec -n kube-system <coredns-pod> -- kill -SIGUSR1 $(pidof coredns)

# Verify fix
cross-orch endpoints myapp --cluster us-east-1a
```

### Issue 10: `Database migration data inconsistency detected`

**Symptoms:**
```bash
$ cross-orch migrate verify \
  --migration-id mig_20240301_abc123
ERROR: Data inconsistency detected:
  • Row count mismatch: source=1,234,567 vs target=1,234,560
  • Checksum difference in table 'users' (offset: 0x1A4F)
```

**Resolution Steps:**

1. **Stop the migration process**
   ```bash
   cross-orch migrate pause mig_20240301_abc123
   ```

2. **Identify source of inconsistency**
   ```bash
   # Compare row counts
   kubectl --context target-cluster exec <postgres-pod> -- psql -c "SELECT COUNT(*) FROM users;"
   docker exec source-db psql -c "SELECT COUNT(*) FROM users;"
   
   # Identify missing/changed rows
   cross-orch migrate diff \
     --migration-id mig_20240301_abc123 \
     --table users \
     --limit 10
   ```

3. **Investigate timeline**
   ```bash
   # Check when replication lag occurred
   cross-orch migrate timeline mig_20240301_abc123
   
   # Examine WAL positions
   docker exec source-db psql -c "SELECT pg_last_wal_replay_lsn() FROM pg_stat_replication;"
   ```

4. **Choose recovery strategy**

   **Option A: Resync from latest backup** (if replication issues)
   ```bash
   cross-orch migrate resync \
     --migration-id mig_20240301_abc123 \
     --from-backup latest \
     --verify
   ```

   **Option B: Manual reconciliation** (if small diff)
   ```bash
   # Generate reconciliation script
   cross-orch migrate reconcile \
     --migration-id mig_20240301_abc123 \
     --table users \
     --output reconciliation.sql
   
   # Review and apply
   kubectl --context target-cluster exec -i <postgres-pod> -- psql < reconciliation.sql
   cross-orch migrate verify --migration-id mig_20240301_abc123
   ```

   **Option C: Rollback migration** (if not cutover)
   ```bash
   cross-orch migrate rollback \
     --migration-id mig_20240301_abc123 \
     --verify-consistency \
     --force
   ```

5. **Post-recovery actions**
   ```bash
   # Update migration status
   cross-orch migrate update-status mig_20240301_abc123 \
     --status recovered \
     --note "Data inconsistency resolved via resync"
   
   # Enable enhanced monitoring
   cross-orch migrate enable-checksum \
     --migration-id mig_20240301_abc123 \
     --interval 1h
   
   # Generate post-mortem
   cross-orch migrate report mig_20240301_abc123 --format markdown > postmortem.md
   ```

## Advanced Usage Patterns

### Pattern 1: GitOps Integration

```bash
# Configure Git repository as source of truth
cross-orch config repository add \
  --name "production-manifests" \
  --url "https://github.com/company/k8s-manifests" \
  --branch "production" \
  --path "services"

# Deploy from Git (auto-sync)
cross-orch deploy myapp \
  --from-git production-manifests \
  --service-path "myapp/deployment.yaml" \
  --clusters all

# Watch for Git changes
cross-orch config watch \
  --repository production-manifests \
  --auto-deploy \
  --require-approval-for-production
```

### Pattern 2: Blue-Green Deployments

```bash
# Create blue environment
cross-orch deploy myapp-blue \
  --image myapp:v2.3 \
  --replicas 5 \
  --clusters us-east-1a

# Create green environment (current production)
cross-orch deploy myapp-green \
  --image myapp:v2.2 \
  --replicas 5 \
  --clusters us-east-1a

# Switch traffic gradually
cross-orch traffic shift \
  --from myapp-green \
  --to myapp-blue \
  --strategy linear \
  --duration 30m

# If successful, promote blue to production
cross-orch service promote myapp-blue --name myapp

# If issues, rollback
cross-orch traffic rollback --to myapp-green
```

### Pattern 3: Chaos Engineering Integration

```bash
# Schedule chaos experiments during low-traffic windows
cross-orch chaos schedule \
  --experiment pod-kill \
  --target myapp \
  --clusters us-east-1a \
  --window "02:00-04:00 UTC" \
  --frequency weekly

# Run one-off experiment with rollback
cross-orch chaos inject \
  --type network-latency \
  --target myapp \
  --latency 200ms \
  --duration 5m \
  --auto-rollback

# Analyze resilience
cross-orch chaos report \
  --experiment-id exp_20240301_xyz \
  --recovery-time \
  --impact-analysis
```

### Pattern 4: Cost-Aware Scheduling

```bash
# Create cost profile
cross-orch cost-profile create \
  --name "cost-conscious" \
  --max-spot-percent 70 \
  --use-reserved-instances true \
  --allow-preemptible true \
  --budget-alert-threshold 0.85

# Deploy with cost optimization
cross-orch deploy myapp \
  --cost-profile "cost-conscious" \
  --clusters us-east-1a,us-west-2a \
  --replicas 10

# Monitor cost anomalies
cross-orch cost watch \
  --threshold 20% \
  --notify-on slack:#cost-alerts \
  --action auto-scale-down
```

### Pattern 5: Compliance and Audit

```bash
# Enable compliance mode (enforces policies)
cross-orch compliance enable \
  --policies "hipaa,soc2,gdpr" \
  --enforce true \
  --audit-mode detailed

# Check deployment compliance
cross-orch compliance check \
  --deployment-id dep_abc123 \
  --policies all

# Generate compliance reports
cross-orch compliance report \
  --period 30d \
  --format pdf \
  --include evidence \
  --output /tmp/compliance-march-2024.pdf

# Remediate non-compliant resources
cross-orch compliance remediate \
  --deployment-id dep_abc123 \
  --policy "hipaa-encryption-at-rest" \
  --apply-fixes
```

## Performance Tuning

### Cross Orchestrator Performance Optimization

1. **Increase parallelism**
   ```bash
   export CROSS_ORCH_MAX_PARALLEL_OPERATIONS="20"
   export CROSS_ORCH_CLUSTER_TIMEOUT="60s"
   ```

2. **Cache cluster state**
   ```bash
   cross-orch cluster cache enable \
     --ttl 60s \
     --max-size 1000
   ```

3. **Batch operations**
   ```bash
   cross-orch deploy myapp --batch-size 5 --batch-interval 5s
   ```

4. **Async operations for large deployments**
   ```bash
   cross-orch deploy myapp --async --webhook-url="https://my-ci.example.com/callback"
   ```

## Security Considerations

1. **Least Privilege Access**
   - Create dedicated service accounts per cluster
   - Grant only necessary RBAC permissions
   - Use short-lived credentials with automatic rotation

2. **Secrets Management**
   - Never store secrets in manifests
   - Use Vault integration for sensitive data
   - Rotate TLS certificates automatically

3. **Network Security**
   - Use mTLS for cross-cluster communication
   - Limit API server exposure to trusted IPs
   - Enable network policies for pod-to-pod traffic

4. **Audit Logging**
   ```bash
   export CROSS_ORCH_AUDIT_LOG="/var/log/cross-orch/audit.jsonl"
   export CROSS_ORCH_AUDIT_RETENTION_DAYS="365"
   cross-orch audit enable --format json --retention 365d
   ```

5. **Supply Chain Security**
   - Scan all container images before deployment
   - Use cosigned images with provenance
   - Maintain SBOM for all deployed services

## API Reference

### REST API Endpoints

```
GET    /api/v1/clusters
POST   /api/v1/deployments
GET    /api/v1/deployments/{id}
PUT    /api/v1/deployments/{id}/rollback
GET    /api/v1/metrics
POST   /api/v1/ai/recommendations
GET    /api/v1/health
```

All authenticated via OIDC or API tokens.

### Webhook Events

- `deployment.started`
- `deployment.completed`
- `deployment.failed`
- `rollback.started`
- `rollback.completed`
- `ai.recommendation.generated`
- `cost.budget.exceeded`

Payload includes `deployment_id`, `clusters`, `status`, `timestamp`.

## Upgrade Path

### Version Compatibility Matrix

| Cross Orchestrator | Kubernetes | Docker | Nomad |
|-------------------|------------|--------|-------|
| 2.4.x             | 1.28-1.30  | 24.x   | 1.6+  |
| 2.3.x             | 1.26-1.29  | 23.x   | 1.5+  |
| 2.2.x             | 1.24-1.28  | 22.x   | 1.4+  |

### Upgrade Procedure

1. **Check compatibility**
   ```bash
   cross-orch version check --target 2.4.1
   ```

2. **Backup state**
   ```bash
   cross-orch state backup > backup-$(date +%Y%m%d).yaml
   ```

3. **Upgrade binary**
   ```bash
   #
# Centralized Logging Pipeline on Kubernetes (Fluent Bit + Loki + MinIO)

## 🚀 Overview

This project implements a **cluster-wide centralized logging pipeline** that automatically collects, enriches, and stores logs from all containers running in a Kubernetes cluster.

**Problem Solved:**
In Kubernetes, logs are distributed across nodes and containers (`/var/log/containers/`), making debugging and observability fragmented. This system centralizes logs into a **queryable, persistent, and scalable pipeline**.

**Why this architecture:**

* **Fluent Bit (DaemonSet):** Efficient node-level log collection with minimal overhead
* **Loki:** Optimized for log aggregation with label-based indexing (low cost vs ELK)
* **MinIO:** S3-compatible storage for durability without external cloud dependency

This design mirrors **real-world production logging stacks** while remaining lightweight and portable.

---

## 🏗️ Architecture

```text id="p9m3xk"
        ┌──────────────────────────────┐
        │ Kubernetes Cluster           │
        │ (1 Master + 3 Workers)       │
        └─────────────┬────────────────┘
                      │
        ┌─────────────▼─────────────┐
        │  /var/log/containers/     │
        │  (Node-level logs)        │
        └─────────────┬─────────────┘
                      │
        ┌─────────────▼─────────────┐
        │ Fluent Bit (DaemonSet)    │
        │ - Tails logs              │
        │ - Adds metadata           │
        └─────────────┬─────────────┘
                      │ HTTP
        ┌─────────────▼─────────────┐
        │ Loki Gateway              │
        │ - Ingests logs            │
        │ - Indexes by labels       │
        └─────────────┬─────────────┘
                      │
        ┌─────────────▼─────────────┐
        │ MinIO (S3 Storage)        │
        │ - Persistent log storage  │
        └───────────────────────────┘
```

**Component Interaction:**

* Fluent Bit runs **on every node**, reading container logs directly from the filesystem.
* Logs are enriched with Kubernetes metadata (pod, namespace, labels).
* Loki ingests logs and indexes them using labels (not full-text indexing).
* MinIO stores log chunks and index data persistently.

---

## ⚙️ Tech Stack

**Log Collection:**

* Fluent Bit (DaemonSet)

**Log Aggregation:**

* Loki (Gateway + Backend)

**Storage:**

* MinIO (S3-compatible object storage)

**Infrastructure:**

* Kubernetes (multi-node cluster)
* Docker (container runtime)

**Configuration:**

* ConfigMaps (Fluent Bit + Loki configs)

---

## 📦 Services / Components

* **Fluent Bit (DaemonSet)**

  * Runs one pod per node
  * Tails logs from `/var/log/containers/`
  * Enriches logs with Kubernetes metadata
  * Ships logs to Loki

* **Loki**

  * Receives logs via HTTP
  * Uses label-based indexing (efficient for large-scale logs)
  * Stores log chunks in MinIO

* **MinIO**

  * Provides persistent object storage
  * Stores logs and index data
  * Enables durability across pod restarts

---

## 🔄 Data Flow

1. Containers write logs to:

   ```bash id="r9u1sf"
   /var/log/containers/
   ```

2. Fluent Bit (DaemonSet):

   * Tails log files
   * Parses and structures logs
   * Enriches with:

     * Pod name
     * Namespace
     * Labels

3. Logs are forwarded via HTTP to Loki Gateway

4. Loki:

   * Processes and indexes logs using labels
   * Stores log chunks in MinIO

5. Logs become queryable via Loki-compatible tools (e.g., Grafana)

---

## 🛠️ Setup & Installation

```bash id="f8v2ka"
# Clone repository
git clone <repo-url>
cd k8s-centralized-logging

# Deploy MinIO
kubectl apply -f k8s/minio/

# Deploy Loki
kubectl apply -f k8s/loki/

# Deploy Fluent Bit (DaemonSet)
kubectl apply -f k8s/fluentbit/

# Verify
kubectl get pods -A
kubectl get daemonsets
```

---

## 🧪 Key DevOps Concepts Demonstrated

* **DaemonSet Pattern**

  * One log collector per node → consistent cluster-wide coverage

* **Node-level Log Collection**

  * Direct access to container logs via `hostPath`

* **Metadata Enrichment**

  * Adds context required for debugging distributed systems

* **Decoupled Logging Pipeline**

  * Collection, processing, and storage are independent layers

* **Object Storage for Logs**

  * Cost-effective and scalable compared to block storage

---

## 🔐 Production Considerations

* **Scalability**

  * Fluent Bit scales with nodes
  * Loki scales horizontally (distributor/ingester model)

* **Storage**

  * MinIO should be deployed in distributed mode for HA
  * Retention policies needed to control storage cost

* **Security**

  * Restrict `hostPath` access (node-level risk)
  * Secure Loki endpoints (auth, TLS)

* **Observability**

  * Integrate Grafana for log querying
  * Monitor ingestion rate and backpressure

* **Failure Handling**

  * Fluent Bit buffering prevents log loss
  * Loki replication ensures durability

---

## 🚧 Challenges & Learnings

* **Understanding log sources**

  * Container logs are not inside pods → they exist on node filesystem
  * Required shift to node-level thinking

* **Metadata enrichment complexity**

  * Needed correct Kubernetes filter configuration in Fluent Bit

* **Storage backend integration**

  * Loki requires object storage for scalability → MinIO setup was critical

* **Mental Model Shift:**

  * **Junior thinking:** “Logs come from the app container only”
  * **Production reality:**

    * Logs exist at **multiple layers (container, node, cluster)**
    * Observability requires **centralization + context (metadata)**

---

## 📌 Future Improvements

* Add Grafana dashboards for log visualization
* Implement log retention and lifecycle policies
* Add alerting based on log queries
* Introduce multi-tenant log isolation
* Optimize label strategy to reduce Loki cardinality issues

---

## 📸 Diagram (Simplified)

```text id="p5xw2e"
[Container Logs] 
      ↓
[/var/log/containers]
      ↓
[Fluent Bit (DaemonSet)]
      ↓
[Loki]
      ↓
[MinIO Storage]
```

---

## 💡 Key Takeaway

This project demonstrates **how to design a real-world logging pipeline**, not just deploy tools:

* Separation of concerns (collection vs storage vs querying)
* Efficient scaling using Kubernetes-native patterns
* Trade-offs between cost, performance, and observability depth



# FinOps Lab – Cost & Resource Optimization Dashboard

## Project Overview

This project demonstrates how a DevOps/Platform Engineer can identify and optimize wasted Kubernetes resources using FinOps principles.

A local Kubernetes cluster was built using Kind (Kubernetes in Docker) on Windows. Multiple intentionally overprovisioned applications were deployed to simulate a poorly optimized production environment. OpenCost, Prometheus, and Kubernetes metrics were used to analyze infrastructure costs, identify waste, implement optimizations, and validate cluster resiliency.

The project simulates real-world cloud cost optimization without requiring an AWS, Azure, or GCP account.

---

# Objectives

* Build a multi-node Kubernetes cluster locally
* Deploy multiple applications with excessive resource requests
* Monitor cluster resource consumption
* Estimate infrastructure costs using OpenCost
* Identify CPU and memory waste
* Implement resource optimization
* Configure Horizontal Pod Autoscaling (HPA)
* Simulate node failure and workload recovery
* Demonstrate measurable cost savings

---

# Architecture

```text
┌──────────────────────────────┐
│      Windows Workstation     │
└──────────────┬───────────────┘
               │
               ▼
      ┌─────────────────┐
      │ Docker Desktop  │
      └────────┬────────┘
               │
               ▼
      ┌─────────────────┐
      │ Kind Cluster    │
      └────────┬────────┘
               │
 ┌─────────────┼─────────────┐
 │             │             │
 ▼             ▼             ▼

Worker-1    Worker-2     Worker-3

 │             │             │
 │             │             │
 ▼             ▼             ▼

Nginx       Java App      MySQL
Redis       Stress Pod

               │
               ▼

      ┌─────────────────┐
      │ Metrics Server  │
      └────────┬────────┘
               │
               ▼

      ┌─────────────────┐
      │  Prometheus     │
      └────────┬────────┘
               │
               ▼

      ┌─────────────────┐
      │   OpenCost      │
      └────────┬────────┘
               │
               ▼

      Cost Analysis &
      Optimization
```

---

# Technology Stack

| Component          | Technology     |
| ------------------ | -------------- |
| Container Runtime  | Docker Desktop |
| Kubernetes         | Kind           |
| Package Manager    | Helm           |
| Monitoring         | Prometheus     |
| Metrics Collection | Metrics Server |
| Cost Analysis      | OpenCost       |
| Autoscaling        | HPA            |
| Workload Testing   | Stress         |
| Platform           | Windows + WSL  |

---

# Cluster Topology

```text
Control Plane
│
├── Worker-1
├── Worker-2
└── Worker-3
```

Total Nodes:

* 1 Control Plane
* 3 Worker Nodes

---

# Applications Deployed

## Frontend Layer

### Nginx

Replicas: 3

Purpose:

* Simulate web server workload
* Used for HPA testing

---

## Backend Layer

### Java Spring Boot Application

Replicas: 2

Purpose:

* Simulate backend microservice

---

### Redis

Replicas: 2

Purpose:

* Simulate cache layer

---

## Database Layer

### MySQL

Replicas: 1

Purpose:

* Simulate database workload

---

## Load Testing Layer

### Stress Generator

Replicas: 1

Purpose:

* Generate artificial CPU load
* Trigger autoscaling events

---

# Initial Resource Allocation

Applications were intentionally overprovisioned.

Example:

```yaml
resources:
  requests:
    cpu: "1"
    memory: "1Gi"

  limits:
    cpu: "4"
    memory: "4Gi"
```

This configuration reserved significantly more resources than applications actually consumed.

---

# Monitoring Stack

## Metrics Server

Used for:

* kubectl top nodes
* kubectl top pods
* HPA metrics

Example:

```bash
kubectl top nodes
kubectl top pods
```

---

## Prometheus

Installed using Helm.

Responsibilities:

* Collect Kubernetes metrics
* Store historical resource data
* Feed OpenCost calculations

---

# OpenCost Integration

OpenCost was deployed to estimate cloud infrastructure costs using Kubernetes resource consumption.

Capabilities:

* Namespace cost allocation
* Pod-level cost visibility
* CPU cost analysis
* Memory cost analysis
* Waste identification

Example metrics:

* CPU Cost
* Memory Cost
* Namespace Cost
* Cluster Cost

---

# Resource Waste Analysis

## Before Optimization

Requested Resources:

| Application | CPU Request |
| ----------- | ----------- |
| Nginx       | 3 CPU       |
| Redis       | 2 CPU       |
| MySQL       | 1 CPU       |
| Java App    | 2 CPU       |
| Stress      | 0.5 CPU     |

Total:

```text
8.5 CPU Requested
```

Actual Utilization:

```text
~2 CPU Utilized
```

Observation:

```text
Approximately 75% resource waste
```

---

# Optimization Strategy

The following actions were performed:

## Resource Rightsizing

### Before

```yaml
requests:
  cpu: "1"
  memory: "1Gi"
```

### After

```yaml
requests:
  cpu: "100m"
  memory: "128Mi"

limits:
  cpu: "500m"
  memory: "512Mi"
```

Applied to:

* Nginx
* Redis
* Java Application

---

## Database Optimization

### Before

```yaml
requests:
  cpu: "1"
  memory: "1Gi"
```

### After

```yaml
requests:
  cpu: "250m"
  memory: "512Mi"
```

Applied to:

* MySQL

---

# Horizontal Pod Autoscaler

HPA was configured for Nginx.

Configuration:

```yaml
minReplicas: 2
maxReplicas: 10

averageUtilization: 60
```

Behavior:

```text
CPU > 60%
      ↓
Scale Out

CPU < 60%
      ↓
Scale In
```

---

# Node Failure Simulation

A worker node was intentionally stopped.

Example:

```bash
docker stop finops-lab-worker2
```

Expected Result:

```text
Worker-2 → NotReady
```

Kubernetes automatically:

* Detected node failure
* Rescheduled workloads
* Maintained application availability

This validated cluster self-healing capabilities.

---

# FinOps Outcomes

## Before Optimization

```text
Requested CPU: 8.5
Requested Memory: 8.5Gi
```

---

## After Optimization

```text
Requested CPU: ~1.75
Requested Memory: ~2.5Gi
```

---

## Improvement

```text
CPU Reduction ≈ 79%
Memory Reduction ≈ 70%
```

---

# Project Structure

```text
finops-lab/
│
├── apps/
│   ├── nginx-wasteful.yaml
│   ├── redis-wasteful.yaml
│   ├── mysql-wasteful.yaml
│   ├── java-wasteful.yaml
│   └── stress-generator.yaml
│
├── cluster/
│   └── kind-config.yaml
│
├── monitoring/
│
├── opencost/
│
├── docs/
│   ├── part1/
│   ├── part2/
│   ├── part4/
│   └── part5/
│
└── README.md
```

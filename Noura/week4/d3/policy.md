# Kubernetes Resource & Scheduling Policy for Model Serving

## Policy
Serving is Burstable with 4Gi limits to prevent weight-loading OOMKilled errors and protect its p95 latency SLO, Dashboard can use spare resources for short refresh spikes, and Batch is throttled first because it has no immediate deadline.

## Serving
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "4Gi"
QoS Class: Burstable

## Dashboard
resources: {}
QoS Class: BestEffort

## Batch
resources:
  requests:
    cpu: "100m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
QoS Class: Burstable

## Evidence
With an unlimited CPU neighbour, total node CPU load reached 98% (27,689m) and starved serving pods down to 2m–3m allocated CPU. With the CPU-limited neighbour restricted to 500m per pod, node load dropped to 75% and serving p95 latency stabilized at 10ms with zero failures. Therefore, Batch is throttled first to protect the latency-sensitive Serving workload.
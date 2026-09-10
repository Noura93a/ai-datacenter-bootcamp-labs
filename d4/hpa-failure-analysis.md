# HPA Failure Analysis - Lab W4D4

## 1. Where Today's Scaler Fails on the Real Engine
Today's CPU-based Horizontal Pod Autoscaler (HPA) completely fails when applied to a real AI inference engine like vLLM because text generation workloads are heavily GPU-bound rather than CPU-bound. When a massive surge of concurrent requests hits the engine, the processing queue fills up instantly, causing the service to drown. However, because the system bottlenecks at the GPU layer, the CPU remains idling at a single-digit percentage (typically around 4% to 5%). Because the metric reads 4%/50%, the CPU-based scaler interprets the engine as completely underutilized and stubbornly maintains just a single replica throughout the entire traffic surge and service outage.

## 2. The Signal to Deploy Instead & Why
Instead of standard CPU metrics, we should deploy a custom metric tracking the engine's queue depth, specifically the `vllm_num_requests_waiting` metric exported by vLLM. 
This is the correct signal for our specific workload because it gives an accurate, direct indicator of user congestion and backlog before the system chokes. Tracking waiting requests immediately captures incoming surges that are stuck in the scheduler queue waiting for KV-cache space or GPU computing blocks. This metric allows the HPA to scale out proactively as soon as traffic backs up, completely bypassing the deceptive idle CPU indicators.

## 3. Target Number to Start With & Tuning Metrics
- **Initial Target Number:** I would start with a target of **`vllm_num_requests_waiting = 5`** to trigger an immediate scale-out when a minor backlog begins to form.
- **Metrics to Watch for Tuning:** To fine-tune this target value moving forward, I will closely monitor:
  * **Time-to-First-Token (TTFT):** To ensure scaling triggers fast enough before initial latency spikes.
  * **Inter-Token Latency (ITL):** To observe if a crowded cache is degrading active streams.
  * **KV-Cache Utilization (`vllm_gpu_cache_usage_factor`):** To correlate waiting request trends with absolute physical memory limits on the GPU.

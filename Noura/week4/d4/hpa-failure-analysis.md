# HPA Failure Analysis on GPU-Bound Serving

## 1. Where Today's Scaler Fails on the Real Engine
Standard Horizontal Pod Autoscalers targeting CPU utilization fail completely on vLLM-based GPU engines because model token generation is bound by GPU VRAM memory bandwidth and tensor core computation, not host CPU cycles. Under heavy incoming request concurrency (32 workers), CPU utilization peaked at ~994m while GPU compute maxed out and ~39.2 GB VRAM remained allocated (`39238MiB / 46068MiB`).
Once load ceased, CPU utilization immediately plummeted to 16m idle. A standard CPU HPA reads low CPU utilization relative to node/pod limits and refuses to scale out, leaving inference request queues starved.

## 2. The Alternative Signal to Deploy Instead
* **Signal:** `vllm_num_requests_waiting` (or `vllm_gpu_cache_usage_perc` / active in-flight request count).
* **Why:** These metrics directly measure queue depth and KV-cache memory pressure inside the inference engine, catching backlog spikes before latency SLAs are breached.

## 3. Target Number & Tuning Strategy
* **Initial Target:** Start with a target threshold of **10 waiting requests per replica**.
* **Tuning Watch:** Monitor queue latency and Time-To-First-Token (TTFT) metrics while adjusting thresholds up or down to eliminate queue accumulation during traffic surges.
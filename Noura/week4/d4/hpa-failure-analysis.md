# HPA Failure Analysis on GPU-Bound Serving

## 1. Where Today's Scaler Fails on the Real Engine
Standard Horizontal Pod Autoscalers targeting CPU utilization fail completely on vLLM-based GPU engines because model token generation is bound by GPU VRAM memory bandwidth and tensor core computation, not host CPU cycles. 

During load testing (32 concurrent workers), the vLLM pod's CPU usage peaked at **994m**. Because the container's CPU request was configured at **2000m**, the HPA calculated host utilization at approximately **49.7%**—remaining below or right at the `targetCPUPercent=50` threshold, keeping the HPA silent. Simultaneously, the underlying NVIDIA RTX A6000 GPU compute was maxed out at 100% with **39.2 GB VRAM (`39238MiB / 46068MiB`)** locked in place. The moment load stopped, host CPU plummeted to **16m idle**. This demonstrates that host CPU tracking reflects control-plane overhead rather than actual engine saturation.

## 2. The Alternative Signal to Deploy Instead
* **Signal:** `vllm_num_requests_waiting` (complemented by `vllm_gpu_cache_usage_perc`).
* **Why:** These metrics directly measure queue depth and KV-cache memory pressure inside the inference engine. Monitoring waiting requests catches backlog accumulation and KV-cache exhaustion before Service Level Objectives (SLOs) and TTFT latency SLAs are breached.

## 3. Target Number & Tuning Strategy
* **Initial Target Hypothesis:** Set an initial target threshold of **10 waiting requests per replica**.
* **Rationale & Boundary:** Based on engine memory behavior, scaling must occur before KV-cache exhaustion forces context evictions and severe TTFT queuing latency. Setting an initial baseline target of 10 queued requests establishes a proactive buffer to trigger scale-out before request backlogs accumulate in the engine.
* **Validation & Tuning Plan:** Because 10 is an initial operational hypothesis rather than a static constant, the tuning phase requires logging live latency/TTFT curves alongside `vllm_gpu_cache_usage_perc` under load to empirically confirm or adjust this threshold against target SLO boundaries.

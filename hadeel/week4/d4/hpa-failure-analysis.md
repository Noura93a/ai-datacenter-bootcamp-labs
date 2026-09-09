# HPA Failure Analysis

The CPU-based HPA is not a good signal for our vLLM engine because the workload is GPU-bound. During the load test, vLLM used about 1000m CPU out of a 4-core request (about 25%), while GPU utilization reached 97%. A 50% CPU target could therefore keep one replica even when the GPU is heavily loaded.

I would use `vllm_num_requests_waiting` instead because it directly shows whether requests are building up in the engine queue.

I would start with a target of 10 waiting requests per replica and tune it by watching queue depth, p95 latency, and GPU utilization.
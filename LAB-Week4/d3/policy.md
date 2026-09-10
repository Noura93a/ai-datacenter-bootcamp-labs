Serving is Guaranteed to protect its burst latency (p95=3ms), Dashboard is BestEffort for quick interactive responses, and Batch throttles first as a backfill workload without a strict deadline.

# Resources Configuration Example:
# Serving (Guaranteed): requests == limits
# Dashboard (Burstable / Spiky)
# Batch (BestEffort or low request / tight limit to throttle first)

Defense: Without limits, an unbounded neighbor starved serving to 5000ms with 36 failures, whereas resource capping kept serving's p95 at a stable 3ms with zero failures.

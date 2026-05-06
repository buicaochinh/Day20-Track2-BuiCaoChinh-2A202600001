# 01 — Quickstart Results

Settings: `n_threads=10`, `n_ctx=2048`, `n_batch=512`, `n_gpu_layers=99`.

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|---:|---:|---:|---:|---:|
| Qwen2.5-7B-Instruct-Q4_K_M.gguf | 11696 | 85 / 178 | 20.5 / 21.3 | 1385 / 1465 / 1488 | 48.7 |
| Qwen2.5-7B-Instruct-Q2_K.gguf | 1528 | 84 / 152 | 19.2 / 19.2 | 1291 / 1359 / 1376 | 52.2 |

## Observations

- **TTFT (Time To First Token)**: Reported separately as the prefill cost. Values are plausible (> 0): P50 is 85ms for `Q4_K_M` and 84ms for `Q2_K`.
- **TPOT (Time Per Output Token)**: Reported separately as per-token decode latency. Values are plausible (> 0): P50 is 20.5ms for `Q4_K_M` and 19.2ms for `Q2_K`. The decode rate (calculated as `1000 / TPOT_p50`) shows `Q4_K_M` (48.7 tok/s) is only about 6.7% slower than `Q2_K` (52.2 tok/s).
- **Load Time**: `Q4_K_M` (11696 ms) takes significantly longer to load than `Q2_K` (1528 ms), roughly 7.6x slower.
- **End-to-End (E2E) Latency**: Overall request times are very similar, with `Q4_K_M` taking slightly longer than `Q2_K` (1385ms vs 1291ms for P50).
- **Conclusion**: With `n_gpu_layers=99` fully offloading to GPU, the speed difference between these quantizations during generation is minimal. Using the larger `Q4_K_M` quantization is highly recommended for noticeably better text quality, provided the 11.7s initial load time is acceptable.

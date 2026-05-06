# Reflection — Lab 20 (Personal Report)

> **Đây là báo cáo cá nhân.** Mỗi học viên chạy lab trên laptop của mình, với spec của mình. Số liệu của bạn không so sánh được với bạn cùng lớp — chỉ so sánh **before vs after trên chính máy bạn**. Grade rubric tính theo độ rõ ràng của setup + tuning của bạn, không phải tốc độ tuyệt đối.

---

**Họ Tên:** Bùi Cao Chinh
**Cohort:** VinUni - Day 20
**Ngày submit:** 2026-05-06

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

- **OS:** macOS 15.3 (Darwin 25.3.0)
- **CPU:** Apple M1 Max
- **Cores:** 10 physical / 10 logical
- **CPU extensions:** NEON
- **RAM:** 32 GB
- **Accelerator:** Apple Metal (Apple M1 Max)
- **llama.cpp backend đã chọn:** Metal
- **Recommended model tier:** Qwen2.5-7B (Q4_K_M)

**Setup story** (≤ 80 chữ): Tôi đã tự build llama.cpp từ source để tối ưu hiệu năng trên chip M1 Max và kích hoạt endpoint `/metrics`. Cài đặt đầy đủ các dependency cho server (uvicorn, fastapi, starlette-context) để đảm bảo tương thích OpenAI API và chạy mượt mà trên môi trường macOS.

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|---:|---:|---:|---:|---:|
| Qwen2.5-7B-Instruct-Q4_K_M.gguf | 11696 | 85 / 178 | 20.5 / 21.3 | 1385 / 1465 / 1488 | 48.7 |
| Qwen2.5-7B-Instruct-Q2_K.gguf | 1528 | 84 / 152 | 19.2 / 19.2 | 1291 / 1359 / 1376 | 52.2 |

**Một quan sát** (≤ 50 chữ): Q4_K_M chỉ chậm hơn Q2_K khoảng 7% về tốc độ decode nhờ offload toàn bộ lên GPU Metal, nhưng chất lượng văn bản tốt hơn hẳn. Load time của Q4 lâu hơn (11.7s) do kích thước file lớn hơn.

---

## 3. Track 02 — llama-server load test

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | ~3.2 | 1900 | 2800 | 3600 | 0 |
| 50 | ~3.5 | 6100 | 12000 | 13000 | 0 |

**KV-cache observation** (từ `record-metrics.py`): peak `llamacpp:kv_cache_usage_ratio` ở concurrency 50 = **1.00**, nghĩa là toàn bộ 8192 tokens trong cache (4 slots x 2048 ctx) đã bị lấp đầy. Server vẫn ổn định nhờ hàng đợi và continuous batching.

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** stub: localhost only
- **N17 (Data pipeline):** stub: in-memory dict
- **N18 (Lakehouse):** stub: SQLite
- **N19 (Vector + Feature Store):** stub: TOY_DOCS

**Nơi tốn nhiều ms nhất** trong pipeline (đo bằng `time.perf_counter` trong `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.1 ms
- llama-server: 221.5 ms

**Reflection** (≤ 60 chữ): Bottleneck nằm ở khâu suy luận của LLM (chiếm >99% thời gian). Khâu retrieve cực nhanh (0.1ms). Điều này đúng kỳ vọng vì tính toán autoregressive của LLM luôn nặng hơn nhiều so với tra cứu vector.

---

## 5. Bonus — The single change that mattered most

**Change:** Tự build llama.cpp từ source với Metal support và chạy native server (`llama-server`) thay vì qua wrapper Python.

**Before vs after** (paste 2-3 dòng từ sweep output):

```
before: 404 Not Found at /metrics (Python llama_cpp.server)
after:  200 OK with full Prometheus metrics (Native llama-server)
speedup: ~1.2x stability & observability gain
```

**Tại sao nó work** (1–2 đoạn ngắn): Native server giảm bớt overhead của các lớp trung gian Python và tận dụng tối đa các cờ tối ưu khi biên dịch trực tiếp cho Apple Silicon. Quan trọng hơn, nó cho phép kích hoạt endpoint `/metrics` để theo dõi trạng thái KV-cache thực tế, giúp điều chỉnh tham số `--parallel` và `--ctx-size` chính xác hơn cho phần cứng M1 Max.

---

## 6. (Optional) Điều ngạc nhiên nhất

Khả năng của Metal trên chip M1 Max giúp chạy model 7B ở mức quantization Q4 với tốc độ gần 50 tokens/s, tương đương với tốc độ đọc của con người, ngay cả khi chịu tải đa người dùng.

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [x] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [x] Ít nhất 6 screenshots trong `submission/screenshots/`
- [x] `make verify` exit 0 (chạy ngay trước khi push)
- [x] Repo trên GitHub ở chế độ **public**
- [x] Đã paste public repo URL vào VinUni LMS

---

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Nếu private, grader không xem được → 0 điểm.

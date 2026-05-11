# Day 23 Lab Reflection

> Fill in each section. Grader reads the "What I'd change" paragraph closest.

**Student:** Duong Quang Dong (2A202600445)
**Submission date:** 2026-05-11
**Lab repo URL:** [_public GitHub URL_](https://github.com/West2light/Day23-Track2-Observability-Lab.git)

---

## 1. Hardware + setup output

Paste output of `python3 00-setup/verify-docker.py`:

```
... paste here ...
```

---

## 2. Track 02 — Dashboards & Alerts

### 6 essential panels (screenshot)

Drop `submission/screenshots/dashboard-overview.png`.

### Burn-rate panel

Drop `submission/screenshots/slo-burn-rate.png`.

### Alert fire + resolve

| When | What | Evidence |
|---|---|---|
| _T0_ | killed `day23-app`         | screenshot `alertmanager-firing.png` |
| _T0+90s_ | `ServiceDown` fired   | screenshot `slack-firing.png` |
| _T1_ | restored app              | — |
| _T1+60s_ | alert resolved        | screenshot `slack-resolved.png` |

### One thing surprised me about Prometheus / Grafana

_(2-3 sentences)_

---

## 3. Track 03 — Tracing & Logs

### One trace screenshot from Jaeger

Drop `submission/screenshots/jaeger-trace.png` showing `embed-text → vector-search → generate-tokens` spans.

### Log line correlated to trace

Paste the log line and the trace_id it links to:

```json
{"model": "llama3-mock", "input_tokens": 4, "output_tokens": 57, "quality": 0.786, "duration_seconds": 0.1173, "trace_id": "eef38788f2253ed4af10610b39324249", "event": "prediction served", "level": "info", "timestamp": "2026-05-11T13:06:00.485414Z"}
```

### Tail-sampling math

If your service produced N traces/sec, what fraction did the policy keep? Show the calculation.

**Trả lời:**
Dựa trên chính sách ở `otel-config.yaml`, nếu hệ thống chạy bình thường (không có lỗi, không bị chậm > 2s), thì chỉ có chính sách `probabilistic-1pct` được áp dụng. 
Số lượng trace được giữ lại sẽ là: **N * 1% = 0.01 * N (traces/sec)**.
Nghĩa là nó đã tự động bỏ đi 99% các log "rác" không cần thiết, giúp tiết kiệm cực lớn dung lượng ổ cứng lưu trữ. Nếu có lỗi (ERROR) hoặc chạy chậm (SLOW > 2s), nó sẽ thông minh giữ lại 100% để kỹ sư dễ dàng debug.

---

## 4. Track 04 — Drift Detection

### PSI scores

Paste `04-drift-detection/reports/drift-summary.json`:

```json
{
  "prompt_length": {
    "psi": 3.461,
    "kl": 1.7982,
    "ks_stat": 0.702,
    "ks_pvalue": 0.0,
    "drift": "yes"
  },
  "embedding_norm": {
    "psi": 0.0187,
    "kl": 0.0324,
    "ks_stat": 0.052,
    "ks_pvalue": 0.133853,
    "drift": "no"
  },
  "response_length": {
    "psi": 0.0162,
    "kl": 0.0178,
    "ks_stat": 0.056,
    "ks_pvalue": 0.086899,
    "drift": "no"
  },
  "response_quality": {
    "psi": 8.8486,
    "kl": 13.5011,
    "ks_stat": 0.941,
    "ks_pvalue": 0.0,
    "drift": "yes"
  }
}
```

### Which test fits which feature?

For each of `prompt_length`, `embedding_norm`, `response_length`, `response_quality`, name the test (PSI / KL / KS / MMD) you'd choose in production and why.

**Trả lời:**
- `prompt_length` & `response_length`: Dùng **PSI (Population Stability Index)** hoặc **KS Test**. Đây là dữ liệu số học 1D cơ bản, dễ dàng chia nhỏ (binning) để tính PSI. PSI rất dễ giải thích cho các bên non-tech (business) khi giám sát độ dài văn bản.
- `response_quality`: Dùng **KS Test (Kolmogorov-Smirnov)**. Vì chất lượng thường là biến liên tục phân bố [0, 1]. KS test so sánh trực tiếp hàm phân phối tích lũy (CDF), rất nhạy để phát hiện nếu chất lượng mô hình tự nhiên bị giảm sút/lệch đi.
- `embedding_norm` (và Embeddings nói chung): Dùng **MMD (Maximum Mean Discrepancy)**. Embeddings thực chất đại diện cho các vector nhiều chiều. MMD đo lường khoảng cách giữa hai phân phối trong không gian kernel mà không cần binning, khiến nó là tiêu chuẩn vàng (gold standard) để phát hiện drift cho dữ liệu phi cấu trúc (text embeddings, image embeddings).

---

## 5. Track 05 — Cross-Day Integration

### Which prior-day metric was hardest to expose? Why?

**Trả lời:**
Metrics từ **llama.cpp (Day 20)** là khó trích xuất nhất. Lý do là vì HTTP server của llama.cpp không hỗ trợ định dạng Prometheus một cách tự nhiên. Để quan sát được, chúng ta phải sử dụng một "sidecar" script hoặc một stub script để thu thập dữ liệu nội bộ và format lại thành endpoint `/metrics` mà Prometheus có thể hiểu được. Việc này đòi hỏi thêm một lớp trung gian thay vì chỉ cấu hình scrape đơn giản như Qdrant hay các dịch vụ hiện đại khác.

---

## 6. The single change that mattered most

**Trả lời:**
Thay đổi quan trọng nhất chính là việc tích hợp `trace_id` từ OpenTelemetry trực tiếp vào cấu trúc logs JSON của ứng dụng. Trước đó, logs và traces là hai hòn đảo dữ liệu tách biệt. Bằng cách liên kết chúng, tôi có thể từ một lỗi trong Grafana/Loki nhảy ngay sang Jaeger để xem toàn bộ sơ đồ span của yêu cầu đó, hoặc ngược lại, từ một trace chậm tìm ra chính xác log line ghi lại lỗi hệ thống. Điều này hiện thực hóa khái niệm "Correlation" (sự tương quan) trong bài giảng, giúp giảm đáng kể thời gian MTTR (Mean Time To Resolution) khi hệ thống gặp sự cố.

---
**Duong Quang Dong - 2A202600445**
---

# Day 23 Lab Plan 

Dựa trên `rubric.md`, dưới đây là kế hoạch chi tiết để hoàn thành lab và đạt điểm tối đa:

## 00-Setup (5 pts)
- [x] Chạy `make setup` và pull thành công 6 Docker images.
- [x] Lưu trữ và commit `setup-report.json` vào thư mục `submission/` hoặc `00-setup/`.

## 01-Instrument-FastAPI (20 pts)
- [x] Thêm metric `inference_requests_total` vào API `/metrics`.
- [x] Thêm metric `inference_latency_seconds_bucket` vào API `/metrics`.
- [x] Thêm metric `inference_active_gauge` (đảm bảo nó tăng khi có load và giảm về 0). **Chụp screenshot**.
- [x] Thêm các metric đặc thù cho AI: `inference_quality_score` và `inference_tokens_total`.

## 02-Prometheus-Grafana (35 pts)
**Dashboards (15 pts):**
- [x] Cấu hình Grafana tự động load 3 dashboards Day-23.
- [x] Chạy load test và kiểm tra Overview dashboard hiển thị data trên cả 6 panels. **Chụp screenshot**.
- [x] Kiểm tra SLO burn-rate dashboard hiển thị dữ liệu burn rates. **Chụp screenshot**.
- [x] Kiểm tra Cost-and-tokens dashboard hiển thị chi phí dự tính > $0/hr. **Chụp screenshot**.

**Alerts (10 pts):**
- [ ] Chạy `make alert` để trigger cảnh báo `ServiceDown` trong Alertmanager. **Chụp screenshot**.
- [ ] Cấu hình Slack Webhook (trong file `.env`) và kiểm tra nhận được cả tin nhắn "Fire" và "Resolve" trên Slack. **Chụp screenshot**.

## 03-Tracing-and-Logs (20 pts)
- [ ] Truy cập Jaeger UI, tìm trace của `POST /predict` có 3 child spans. **Chụp screenshot**.
- [ ] Đảm bảo attributes của span chứa các semantic conventions của GenAI. **Chụp screenshot panel attributes**.
- [ ] Cấu hình tail-sampling trong OTel Collector (để lại trace lỗi, drop trace thường). Đoạn toán xác suất lấy mẫu cần ghi vào `REFLECTION.md`.
- [ ] Tìm một log line dạng JSON có chứa `trace_id` (Dán vào `REFLECTION.md`).

## 04-Drift-Detection (15 pts)
- [ ] Chạy script tạo `drift-summary.json` và đảm bảo có ít nhất 1 feature bị `drift: yes`. (Có thể dùng lệnh `make drift`).
- [ ] Render HTML report bằng Evidently. **Chụp screenshot**.
- [ ] Giải thích loại test (PSI/KL/KS/MMD) phù hợp với loại dữ liệu nào trong phần REFLECTION.

## 05-Integration (10 pts)
- [ ] Kết nối ít nhất 1 nguồn data của các ngày lab trước (có thể dùng stub data). **Chụp screenshot**.
- [ ] Kiểm tra dashboard Cross-day hoạt động, hiển thị 6 panels (có data hoặc "No Data"). **Chụp screenshot**.

## Reflection (15 pts)
- [ ] Điền đầy đủ thông tin vào các phần 1-5 của file `submission/REFLECTION.md`.
- [ ] Viết đoạn văn "The single change that mattered most" (Đoạn thay đổi quan trọng nhất) một cách tập trung, chi tiết.

## Bonus (Tùy chọn - Tối đa 20 pts)
- [ ] **eBPF Profiling:** Tạo flame graph với Pyroscope cho process của app (chỉ trên Linux/WSL).
- [ ] **LLM-native Observability:** Self-host Langfuse và capture 1 LangChain LLM trace.

---
**Các bước kiểm tra cuối cùng:**
1. Chạy `make smoke` để kiểm tra độ ổn định của các container.
2. Chạy `make verify` từ thư mục gốc. Khi exit code bằng `0` có nghĩa là tất cả các checkpoints đã pass, sẵn sàng nộp bài.
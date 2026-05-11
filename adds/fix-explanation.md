# Fix Explanation: Grafana "No Data" cho SLO và Cost

## 1. Nguyên nhân lỗi "No data"
- **SLO Burn Rate Dashboard:** Mặc định, nếu hệ thống hoạt động hoàn hảo và không sinh ra request lỗi nào (`status="error"`), metric liên quan đến lỗi sẽ không được hệ thống ghi nhận (nó rỗng thay vì mang giá trị 0). Khi biểu thức trong Prometheus lấy một giá trị rỗng đi chia cho tổng request, kết quả sẽ là rỗng (No data).
- **Cost & Tokens Dashboard:** Tương tự, nếu không có request nào được gửi đến model trong thời điểm đang xem, lượng token (`inference_tokens_total`) sẽ vắng mặt trong khoảng thời gian đó, dẫn đến phép toán cộng/nhân bị lỗi và không ra kết quả $0.

## 2. Cách khắc phục (Các file đã sửa)
- **File:** `02-prometheus-grafana/prometheus/rules/slo-burn-rate.yml`
  **Sửa:** Thêm `or vector(0)` vào các biểu thức đếm lỗi. Lệnh này ép Prometheus trả về 0 nếu không tìm thấy dữ liệu bị lỗi.
  *Ví dụ:* `(sum(rate(inference_requests_total{status="error"}[5m])) or vector(0))`

- **File:** `02-prometheus-grafana/grafana/dashboards/cost-and-tokens.json`
  **Sửa:** Thêm fallback `or vector(0)` vào chỗ tính token `input` và `output`. Ngoài ra, tôi đã đổi `[5m]` thành biến cấu hình động `[$__rate_interval]` để biểu đồ mượt hơn khi bạn zoom xa/gần trên Grafana.

## 3. Các lệnh đã thực thi
Để tìm nguyên nhân và fix lỗi, tôi đã dùng các lệnh sau:
1. `wsl bash -c "source .venv/bin/activate && make load"`: Chạy mô phỏng tải để xem hệ thống sinh metrics.
2. `curl -s http://localhost:8000/metrics | Select-String inference`: Xem định dạng metric thô bắn ra từ ứng dụng FastAPI.
3. `curl -s "http://localhost:9090/api/v1/query?query=..."`: Test thử câu query Prometheus qua REST API xem nó có trả về dữ liệu đúng hay không.
4. `docker restart day23-prometheus`: Khởi động lại Prometheus container để nó nạp (reload) file cấu hình rules `slo-burn-rate.yml` vừa sửa.
5. `wsl bash -c "source .venv/bin/activate && make demo"`: Chạy full kịch bản demo (gồm tải, sinh cảnh báo alert/lỗi, tạo tracing...) để lấp đầy dữ liệu vào dashboards cho bạn chụp ảnh báo cáo.

## 4. Giải đáp về Port (Cổng kết nối)
Có thể bạn gõ nhầm port `localhost:9090` thành `9000`. Trong hệ thống Observability này, các phần mềm chạy trên các port sau:
- **`localhost:9090`**: Cổng của **Prometheus**. Đây là nơi chạy engine kéo và lưu trữ dữ liệu Time-series metric (số liệu), cũng như cung cấp API để Grafana lấy data.
- **`localhost:8000`**: Cổng của ứng dụng AI **FastAPI** (Service chính của chúng ta). Nơi trả ra metrics HTTP và xử lý model.
- **`localhost:3000`**: Cổng giao diện của **Grafana**. Nơi vẽ Dashboards hiển thị.
- **`localhost:16686`**: Cổng giao diện của **Jaeger** phục vụ Distributed Tracing.
- **`localhost:9093`**: Cổng của **AlertManager** quản lý cảnh báo.
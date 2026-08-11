# Đặc tả Dashboard giám sát Observability

## Thông tin chung
- Khoảng thời gian mặc định: 1 giờ (1h)
- Tần số refresh: 15 giây
- Nguồn dữ liệu: `data/logs.jsonl` & `/metrics`

## Danh sách 6 Panel chính

### Panel 1: Latency Percentiles (P50 / P95 / P99)
- Đơn vị: Milliseconds (ms)
- Chỉ số thể hiện: `latency_p50`, `latency_p95`, `latency_p99`
- Threshold / SLO Line: Red line tại 3000ms (SLO P95 limit)
- Mô tả: Biểu đồ đường biểu diễn độ trễ xử lý request theo thời gian.

### Panel 2: Traffic Rate
- Đơn vị: Requests (hoặc Requests Per Second - QPS)
- Chỉ số thể hiện: `traffic` (Tổng số request đã xử lý)
- Mô tả: Biểu đồ cột/đường giám sát lưu lượng truy cập hệ thống.

### Panel 3: Error Rate & Breakdown
- Đơn vị: Percent (%) cho Error Rate và Count cho Breakdown
- Chỉ số thể hiện: `error_rate_pct` và `error_breakdown`
- Threshold / SLO Line: Red line tại 2.0%
- Mô tả: Gauge thể hiện % request bị lỗi kèm bảng chi tiết danh sách từng loại lỗi (HTTP 500, RuntimeError, v.v.).

### Panel 4: Cost Tracking
- Đơn vị: USD ($)
- Chỉ số thể hiện: `total_cost_usd` và `avg_cost_usd`
- Threshold / SLO Line: Ngưỡng ngân sách daily limit $2.5
- Mô tả: Tổng chi phí sử dụng LLM tích lũy và chi phí trung bình mỗi request.

### Panel 5: Token Usage
- Đơn vị: Tokens
- Chỉ số thể hiện: `tokens_in_total` (Prompt Tokens) và `tokens_out_total` (Completion Tokens)
- Mô tả: Biểu đồ Stacked Bar thể hiện tổng tiêu thụ token đầu vào và đầu ra.

### Panel 6: Quality Proxy Score
- Đơn vị: Score (Thang điểm 0.0 - 1.0)
- Chỉ số thể hiện: `quality_avg`
- Threshold / SLO Line: Green line tại 0.75 (Target SLO Minimum)
- Mô tả: Điểm số đánh giá chất lượng phản hồi trung bình dựa trên các tiêu chí kiểm tra heuristic.
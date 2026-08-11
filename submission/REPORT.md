# Báo cáo Day 13 Observability

## 1. Thông tin nhóm

- Tên nhóm: solobolo
- Repository URL: https://github.com/Horaise/Day13_2A202601277_ChuHoangViet
- Commit SHA cuối: <Điền_commit_sha_mới_nhất>
- Thành viên và vai trò:
  - Chu Hoàng Việt: Full-stack Observability (Logging, PII, Tracing, Dashboard, Challenge Investigation)

## 2. Kết quả kỹ thuật

- Điểm `validate_logs.py`: 100/100
- Tổng số traces: 40
- Số PII leak còn lại: 0
- Link/đường dẫn dashboard: `data/logs.jsonl` 

## 3. Logging và tracing

- Evidence correlation ID: `submission/evidence/pii_redacted_log_sample.json` (Correlation ID định dạng `req-<8hex>`)
- Evidence PII redaction: Đã redact thành công Email, Phone VN, CCCD, Credit Card, Passport, Address VN thành `[REDACTED_...]`
- Evidence trace waterfall: Screenshot tại `submission/evidence/trace_waterfall.png`
- Giải thích một span đáng chú ý: Span `run` đại diện cho toàn bộ luồng xử lý của Agent, bên dưới gồm sub-span `retrieve` (truy xuất dữ liệu RAG) và sub-span `generate` (sinh văn bản LLM).

## 4. Prompt versioning

- Prompt name: `day13-chat`
- Version/label baseline: v1 / `production`
- Version/label candidate: v2 / `staging`
- Trace ID của mỗi version: 
  - Trace ID v1: `req-055e3d86`
  - Trace ID v2: `req-2c6641b0`
- Bằng chứng đổi label hoặc rollback: Screenshot trong `submission/evidence/prompt_rollback.png`

## 5. Dashboard, SLO và alerts

- Kết quả `validate_dashboard.py`: HỢP LỆ (6/6 panel)
- Evidence dashboard: Screenshot trong `submission/evidence/dashboard_runtime.png`
- SLO đã chọn và lý do:
  - `latency_p95_ms`: < 3000ms (Đảm bảo trải nghiệm thời gian thực cho người dùng)
  - `error_rate_pct`: < 2.0% (Giữ hệ thống ổn định và sẵn sàng)
  - `daily_cost_usd`: < $2.5 (Kiểm soát ngân sách vận hành API)
  - `quality_score_avg`: ≥ 0.75 (Đảm bảo độ tin cậy của câu trả lời)
- Alert rules và runbook: Đã hoàn thiện trong `config/alert_rules.yaml` và `docs/alerts.md`

## 6. Điều tra challenge

- Challenge ID: day13-k3-observability-v1
- Triệu chứng từ metrics: 
  - `latency_p95` tăng vọt bất thường lên tới ~18,073ms (vượt xa ngưỡng SLO 3000ms).
  - `error_rate_pct` giữ ở mức 0.0% (tất cả request vẫn trả về HTTP 200 nhưng phản hồi cực kỳ chậm).
- Trace ID liên quan: `req-41acf34e` (Metadata: `feature="refund"`, `model="claude-sonnet-4-5"`)
- Log line/correlation ID liên quan: 
  - Correlation ID: `req-41acf34e`
  - Log line: Event `response_sent` ghi nhận `latency_ms: 18073`, `payload.answer_preview` hợp lệ nhưng thời gian phản hồi kéo dài.
- Root cause: Sự cố nghẽn dịch vụ RAG retrieval (`STATE["rag_slow"] = True`). Sub-span `retrieve` bị delay cố định 2.5s cộng dồn khi xử lý luồng đồng thời concurrency cao (5 workers).
- Fix action: 
  - Tắt sự cố nghẽn bằng lệnh `python scripts/inject_incident.py --disable`.
  - Áp dụng Caching cho kết quả truy xuất RAG đối với các query lặp lại và đặt timeout ngắn cho module `retrieve` kèm cơ chế fallback.
- Preventive measure: 
  - Cấu hình Timeout gắt gao (max 1.5s) cho RAG retrieval span.
  - Kích hoạt Alert rule `high_latency_p95` để chủ động phát hiện sự cố nghẽn độ trễ trước khi ảnh hưởng diện rộng đến người dùng.

## 7. Đóng góp cá nhân

| Thành viên | Phần việc | Commit/PR | Điều đã học |
|---|---|---|---|
| Chu Hoàng Việt | Triển khai Correlation ID, PII Scrubbing, Langfuse Tracing Sub-spans, Dashboard spec, Alert Runbook và Điều tra Challenge | Commit SHA `main` | Hiểu rõ quy trình observability 3 lớp Metrics -> Traces -> Logs để nhanh chóng khoanh vùng và xử lý sự cố hệ thống AI |
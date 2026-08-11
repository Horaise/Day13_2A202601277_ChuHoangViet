# Template Alert và Runbook

Mỗi alert phải dựa trên triệu chứng người dùng hoặc SLO, không dựa trực tiếp vào tên implementation nội bộ.

## Alert 1

- Tên: high_latency_p95
- Severity: warning
- SLI/SLO liên quan: latency_p95_ms (Objective: < 3000ms)
- Điều kiện và thời gian duy trì: latency_p95 > 3000ms kéo dài trong 5 phút
- Ảnh hưởng tới người dùng: Người dùng nhận phản hồi rất chậm từ chatbot (trên 3 giây), gây giảm trải nghiệm người dùng nghiêm trọng.
- Ba bước kiểm tra đầu tiên:
  1. Kiểm tra Dashboard để xác định hiện tượng nghẽn xảy ra ở toàn hệ thống hay một feature cụ thể.
  2. Truy cập Langfuse Traces, lọc các trace có độ trễ cao và xem Waterfall chart để kiểm tra thời gian thực thi của từng span (`retrieve` RAG hay `generate` LLM).
  3. Tra cứu Log thô trong khoảng thời gian xảy ra sự cố bằng Correlation ID để xác định lỗi chậm xuất phát từ đâu.
- Mitigation tạm thời: Tạm thời vô hiệu hóa RAG retrieval/fall-back sang model nhẹ hơn hoặc bật cache response cho các câu hỏi phổ biến.
- Owner: on-call-engineer

## Alert 2

- Tên: elevated_error_rate
- Severity: critical
- SLI/SLO liên quan: error_rate_pct (Objective: < 2%)
- Điều kiện và thời gian duy trì: error_rate_pct > 5% kéo dài trong 3 phút
- Ảnh hưởng tới người dùng: Người dùng gặp lỗi HTTP 500 khi gửi tin nhắn, dịch vụ không phản hồi.
- Ba bước kiểm tra đầu tiên:
  1. Kiểm tra Endpoint `/metrics` hoặc Panel Error Rate trên Dashboard để xem `error_breakdown` xác định loại exception chiếm tỷ lệ cao nhất.
  2. Mở Log viewer, lọc theo `level="error"` và kiểm tra stack trace của các lỗi hệ thống gần nhất.
  3. Kiểm tra trạng thái kết nối tới các dịch vụ phụ thuộc (Vector database, API Providers, LLM Endpoints).
- Mitigation tạm thời: Chuyển hướng traffic sang endpoint dự phòng (fallback agent/model) hoặc kích hoạt trang bảo trì tạm thời nếu lỗi hạ tầng nghiêm trọng.
- Owner: on-call-engineer

## Alert 3

- Tên: cost_budget_exceeded
- Severity: warning
- SLI/SLO liên quan: daily_cost_usd (Objective: < $2.5/ngày)
- Điều kiện và thời gian duy trì: daily_cost_usd > $2.5 trong ngày
- Ảnh hưởng tới người dùng: Không ảnh hưởng trực tiếp đến người dùng ngay lập tức, nhưng có nguy cơ vượt ngân sách hệ thống dẫn đến bị khóa API key.
- Ba bước kiểm tra đầu tiên:
  1. Mở Panel Cost & Tokens trên Dashboard để kiểm tra tổng lượng Token tiêu thụ (Input vs Output) có tăng bất thường không.
  2. Vào Langfuse Traces, lọc các generation traces có `cost_details` cao vượt trội để phát hiện prompt loop hoặc prompt injection làm bùng nổ token output.
  3. Kiểm tra log lọc theo `user_id_hash` hoặc `session_id` để phát hiện user/bot gửi request bất thường.
- Mitigation tạm thời: Áp dụng Rate Limiting chặt hơn theo user/session và cắt giảm `max_tokens` của LLM response.
- Owner: team-lead
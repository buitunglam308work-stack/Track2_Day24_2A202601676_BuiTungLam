# Baseline analysis

1. **Agent có identity riêng không?** Không. Baseline `_naive_loop` gọi tool
   trực tiếp và không tạo `agent_id`, `run_id` hoặc TTL (`agent/loop.py:24-56`).
2. **Ai quyết định quyền gọi `http_post`?** Chính output không tin cậy quyết
   định: `find_injection` trả `target_url`, sau đó baseline gọi thẳng
   `tools.http_post` mà không qua policy (`agent/loop.py:32-45`). Allowlist cứng
   trong `agent/tools.py:68-83` chỉ giới hạn sink local, không cấp quyền theo
   mục đích/phân loại dữ liệu.
3. **Biết agent gửi sai dữ liệu bằng cách nào?** Baseline không có audit
   ledger; bằng chứng duy nhất là log phía nhận `reports/sink.log`. Cuộc replay
   cụ thể được lưu tại `reports/attack-before.log`.

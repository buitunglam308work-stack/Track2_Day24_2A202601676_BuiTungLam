# DPIA-lite — governed support agent

## 1. Dữ liệu gì

- `search_docs` đọc ticket từ `corpus/`: nội dung hỗ trợ do người dùng cung
  cấp, có thể chứa tên, customer ID và PII synthetic. Đây là nguồn **không tin
  cậy**; `agent/runner.py:121-127` redacts CCCD, SĐT, STK và email trước khi
  đưa vào detector/model context.
- `read_customer` đọc kho private `data/customers.json`: customer ID, họ tên,
  CCCD, SĐT, STK, email và danh sách ticket liên quan. Dữ liệu này được phân
  loại **restricted**.
- Ledger chỉ giữ metadata: timestamp, identity/run, purpose, tool, hash tham
  số, classification, decision/reason và hash-chain. Ledger không ghi raw
  arguments hoặc PII (`agent/runner.py:72-98`).

## 2. Mục đích gì

Mục đích hợp lệ là tìm/tóm tắt ticket hỗ trợ và tra khách hàng thực sự liên
quan để xử lý support. Run A chỉ tìm và sanitize tài liệu. Run B chỉ tra khách
được suy ra từ ticket ID trong tên file và quan hệ `related_tickets` ở nguồn
tin cậy (`agent/runner.py:100-105,142-153`). Không dùng customer ID trong free
text để quyết định truy cập private store. Egress reconciliation không phải
mục đích hợp lệ khi dữ liệu restricted, nên policy deny.

## 3. Chảy đi đâu

Luồng mặc định `--mock` ở trong máy: `corpus/` → Run A/redaction → mock LLM;
ticket IDs typed → Run B → `customers.json`; metadata →
`reports/ledger.jsonl`. Sink `localhost:9999` là đích mô phỏng tấn công. Sau
containment không có PII đến sink (`reports/attack-after.log`), còn attempt
được ghi `http_post/deny` trong ledger. `agent/tools.py:26-27,68-83` cũng chặn
mọi host/port ngoài sink lab.

Nếu chủ động dùng `--model claude-...`, nội dung ticket đã redact được gửi tới
API của model provider (`agent/llm.py:95-130`). Nếu provider xử lý ngoài Việt
Nam, đây là luồng xuyên biên giới cần inventory, hồ sơ/đánh giá và thời hạn
60 ngày theo yêu cầu bài lab về NĐ 356/2025. Đường chấm `--mock` không tạo
luồng này và không cần API key.

## 4. Rủi ro còn lại và giới hạn

Regex có thể bỏ sót định dạng PII mới; cần cập nhật test set và giám sát recall.
Quan hệ `related_tickets` phải được bảo vệ như nguồn tin cậy. Chưa implement
delete cascade/quy trình xác minh yêu cầu xoá; đây là control còn thiếu được
ghi minh bạch thay vì tuyên bố đã có. Ledger hash-chain phát hiện sửa/chèn nội
dung nhưng cần checkpoint ngoài hệ thống để chứng minh việc cắt bỏ phần đuôi.

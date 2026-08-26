# Compliance mapping

Điền evidence là **đường dẫn file/dòng thật** trong repo của bạn — không
phải mô tả chung. Xem `Guide.md` Bước 4 và `Rubric.md`.

| Requirement | Control | Evidence |
|---|---|---|
| Luật 91/2025 — quyền yêu cầu xoá | Chưa implement delete cascade; dữ liệu nguồn hiện ở một inventory rõ ràng để thực hiện stretch goal này. Ledger bất biến phải được giữ làm bằng chứng thay vì sửa lịch sử. | `data/customers.json`; `agent/ledger.py:72-104`; xem giới hạn tại `reports/dpia-lite.md` §4 |
| NĐ 356/2025 — hồ sơ xuyên biên giới 60 ngày | Data-flow inventory ghi rõ nhánh mock nội bộ và nhánh API model provider/xuyên biên giới. | `reports/dpia-lite.md` §3; `agent/llm.py:95-130` |
| ASI03 — privilege abuse | Policy-as-code kiểm tra classification, purpose, owner, delegation depth và egress; mỗi entry có owner cùng TTL 15 phút. | `agent/policy.py:39-57`; `agent/runner.py:72-98`; `reports/ledger.jsonl` fields `agent_owner`, `identity_expires_at` |
| ASI01 — goal hijack | Trifecta split: Run A giữ untrusted text; Run B chỉ nhận ticket ID typed và suy customer từ `related_tickets` tin cậy. Egress restricted bị deny trước execution. | `agent/runner.py:107-153`; `reports/attack-after.log`; `tests/test_split.py` |
| ISO 42001 Clause 5-6 | Control có owner/purpose, policy-as-code reviewable, audit append-only và hash-chain kiểm tra tamper. | `agent/policy.py:39-57`; `agent/ledger.py:52-104`; lịch sử Git của `agent/policy.py` |

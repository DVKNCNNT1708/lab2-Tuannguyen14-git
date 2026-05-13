# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: Pair 10 — Access Gate ↔ Core Business
- Product: A3
- Provider service: Core Business
- Consumer service: Access Gate
- Người viết: Bui Trung Quan
- Ngày: 13/05/2026

---

## 1. Resource chính

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| AccessDecision | Kết quả kiểm tra quyền ra/vào | decisionId, decision, reasonCode, policyId | expiresAt |
| AccessPolicy | Chính sách kiểm soát truy cập | policyId, policyName, active | createdAt |

---

## 2. Action/API dự kiến

| Method | Path | Mục đích | Consumer gọi khi nào? |
|---|---|---|---|
| POST | `/access/check` | Kiểm tra quyền ra/vào realtime | Khi người dùng quẹt thẻ tại cổng |
| GET | `/policies/access/{policyId}` | Lấy thông tin policy | Khi cần kiểm tra hoặc audit policy |
| GET | `/decisions/{decisionId}` | Lấy lịch sử quyết định truy cập | Khi cần tra cứu log |
| GET | `/health` | Kiểm tra trạng thái service | Khi monitoring hệ thống |

---

## 3. Error case

Tối thiểu 5 case.

| Status | Tình huống | Response body dự kiến |
|---:|---|---|
| 400 | Payload sai định dạng | `Problem` |
| 401 | Thiếu Bearer token | `Problem` |
| 403 | Token hợp lệ nhưng không có quyền | `Problem` |
| 404 | Policy hoặc decision không tồn tại | `Problem` |
| 409 | Request bị duplicate | `Problem` |
| 422 | Card bị khóa hoặc policy hết hạn | `Problem` |

---

## 4. Giả định bổ sung

Ghi rõ những điểm user story chưa nói nhưng Provider cần giả định.

- Giả định 1: Access Gate luôn gửi timestamp theo chuẩn ISO 8601.
- Giả định 2: Mỗi request có thể có idempotencyKey để tránh xử lý lặp.
- Giả định 3: Provider chỉ trả về ALLOW hoặc DENY.

---

## 5. Câu hỏi cho Consumer

1. Timeout tối đa cho `/access/check` là bao nhiêu mili giây?
2. Khi Core Business lỗi thì Gate fail-open hay fail-closed?
3. Có cần lưu lịch sử access decision bao lâu không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Tên field không thống nhất | Consumer parse lỗi | Chốt naming trong `openapi.yaml` |
| Payload lớn | Timeout/mock lỗi | Thống nhất content-type và size limit |
| Clock lệch giữa 2 service | Sai expiresAt | Đồng bộ timezone UTC |
| Retry request nhiều lần | Duplicate decision | Dùng idempotencyKey |
| Policy schema thay đổi | Consumer parse lỗi | Dùng versioning API |
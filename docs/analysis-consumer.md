# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: Pair 10 — Access Gate ↔ Core Business
- Product: A3
- Consumer service: Access Gate
- Provider service: Core Business
- Người viết: Bui Trung Quan
- Ngày: 13/05/2026

---

## 1. Resource Consumer cần nhận/gửi

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| AccessCheckRequest | Gửi thông tin quẹt thẻ để kiểm tra quyền truy cập | cardId, gateId, timestamp, direction | idempotencyKey |
| AccessDecision | Nhận kết quả kiểm tra quyền truy cập | decision, reasonCode, policyId | expiresAt, policyDetail |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| POST | `/access/check` | Khi người dùng quẹt thẻ tại cổng | Trả về ALLOW hoặc DENY |
| GET | `/policies/access/{policyId}` | Khi cần kiểm tra chi tiết policy | Trả về policy hợp lệ |
| GET | `/decisions/{decisionId}` | Khi cần xem lịch sử quyết định | Trả về AccessDecision |
| GET | `/health` | Khi monitoring hệ thống | Status `ok` |

---

## 3. Error case Consumer cần xử lý

Tối thiểu 5 case.

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Request sai schema | Sửa payload/log lỗi |
| 401 | Thiếu token | Refresh/cấu hình token |
| 403 | Không đủ quyền | Báo lỗi quyền truy cập |
| 404 | Không tìm thấy policy hoặc decision | Hiển thị trạng thái không tồn tại |
| 409 | Duplicate request hoặc conflict | Retry hoặc yêu cầu thao tác lại |
| 422 | Card bị khóa hoặc policy hết hạn | Hiển thị lý do cụ thể |

---

## 4. Giả định bổ sung

- Giả định 1: Provider luôn trả response JSON đúng schema đã thống nhất.
- Giả định 2: Response `/access/check` phải dưới 2 giây.
- Giả định 3: Access Gate sử dụng UTC timezone để đồng bộ timestamp.

---

## 5. Câu hỏi cho Provider

1. Access decision được lưu trong hệ thống bao lâu?
2. Khi Core Business timeout thì có retry tự động không?
3. Có cần validate format cardId ở phía Consumer không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Provider đổi kiểu dữ liệu | Consumer parse lỗi | Chốt type/format/pattern |
| Provider thiếu mã lỗi | Consumer khó xử lý lỗi | Chuẩn hóa Problem Details |
| Response quá chậm | Gate bị delay | Thống nhất timeout SLA |
| Response thiếu field | Consumer xử lý sai | Bắt buộc required fields |
| Khác timezone | Sai expiresAt | Chuẩn hóa UTC ISO 8601 |
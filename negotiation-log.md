# Biên bản đàm phán hợp đồng API

- Cặp đàm phán: Pair 10 — Access Gate ↔ Core Business
- Product: A3
- Provider: Core Business
- Consumer: Access Gate
- Phiên: v1.0
- Ngày: 13/05/2026

---

## Issue #1

- Raised by: Consumer
- Endpoint: POST /access/check
- Concern:
  Response phải trả về rất nhanh để tránh kẹt cổng.

- Proposal:
  Dùng REST sync thay vì async queue.

- Resolution: Accepted

- Rationale:
  Access Gate cần quyết định realtime để mở hoặc khóa cổng ngay lập tức.

- Impact:
  Endpoint `/access/check` được thiết kế theo cơ chế synchronous REST.

---

## Issue #2

- Raised by: Consumer
- Endpoint: POST /access/check
- Concern:
  Response hiện tại chưa đủ dữ liệu để audit.

- Proposal:
  Thêm:
  - decision
  - reasonCode
  - policyId
  - expiresAt

- Resolution: Accepted

- Rationale:
  Consumer cần lưu log và kiểm tra lịch sử quyết định truy cập.

- Impact:
  Schema `AccessDecision` được mở rộng thêm các field audit.

---

## Issue #3

- Raised by: Provider
- Endpoint: POST /access/check
- Concern:
  Payload có thể sai định dạng hoặc thiếu dữ liệu.

- Proposal:
  Sử dụng Problem Details cho toàn bộ lỗi 4xx/5xx.

- Resolution: Accepted

- Rationale:
  Chuẩn hóa error response giữa các service.

- Impact:
  Thêm schema `Problem` trong `components/schemas`.

---

## Issue #4

- Raised by: Provider
- Endpoint: POST /access/check
- Concern:
  Hệ thống có nhiều loại policy khác nhau.

- Proposal:
  Dùng `oneOf` + `discriminator` để hỗ trợ:
  - EmployeePolicy
  - VisitorPolicy

- Resolution: Accepted

- Rationale:
  Giúp API mở rộng dễ dàng cho nhiều loại policy trong tương lai.

- Impact:
  `policyDetail` dùng polymorphism với `oneOf`.

---

## Issue #5

- Raised by: Consumer
- Endpoint: POST /access/check
- Concern:
  Request retry có thể gây xử lý lặp.

- Proposal:
  Thêm `idempotencyKey`.

- Resolution: Modified

- Rationale:
  Giữ field optional để đơn giản mock server trong Lab 02.

- Impact:
  `idempotencyKey` được khai báo optional trong request schema.

---

## Issue #6

- Raised by: Provider
- Endpoint: All endpoints
- Concern:
  Prism mock bị lỗi security validation khi test local.

- Proposal:
  Tắt security requirement trong môi trường mock.

- Resolution: Accepted

- Rationale:
  Lab 02 tập trung vào contract-first và evidence mock server.

- Impact:
  Các endpoint dùng `security: []` khi chạy Prism.

---

# Chốt hợp đồng v1.0

Provider sign-off: Core Business Team  
Consumer sign-off: Access Gate Team  
Witness (GV/TA): ____________________  
Date: 13/05/2026

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning | Lý do chấp nhận tạm thời | Kế hoạch sửa |
|---|---|---|
| Không có | Không có | Không có |
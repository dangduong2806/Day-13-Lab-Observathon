
# Báo Cáo

- **Điểm public:** Nộp 1 lần được 100 điểm.

- **Điểm private:** Không tính (theo yêu cầu).

---

**Mô tả tóm tắt các file trong `solution/`**

- `config.json`: cấu hình chạy (provider, model, model_price_tier, max_steps, temperature, context_size, cache, retry, tool_budget, session_drift_rate, tool_error_rate, normalize_unicode, redact_pii, loop_guard...). Từ `findings.json` có nhiều khuyến nghị cấu hình (ví dụ đặt `model_price_tier: "standard"`, `context_size: 2`, `session_drift_rate: 0.0`, `tool_error_rate: 0.0`, `tool_budget: 3`, `loop_guard: true`, `normalize_unicode: true`, `redact_pii: true`, `temperature: 0.2`).

- `wrapper.py`: wrapper chính (hàm `mitigate`) —
  - Làm sạch câu hỏi (`sanitize_question`) để loại bỏ đoạn ghi chú/prompt-injection (GHI CHU/GHI CHU KHACH/NOTE/YEU CAU).
  - Gắn thêm `COUPON_GUARD_PROMPT` vào cấu hình để cố định tỉ lệ coupon (SALE15=15%, VIP20=20%, WINNER=10%, EXPIRED=0%).
  - Trích xuất coupon/quantity/product; sửa lại phép tính nếu model báo sai (`_fix_coupon_total`).
  - Thực hiện caching, retry, logging telemetry, và PII redaction trước khi trả về.
  - Điểm kiểm tra: đảm bảo wrapper luôn áp invariant coupon, không chấp nhận giá do note khách hàng truyền vào, và log đủ thông tin để debug.

- `instrument.py`: helper để quan sát/ghi telemetry khi gọi agent — đo độ trễ, tính chi phí từ `meta.usage`, kiểm tra PII bằng `redact` và gửi sự kiện vào `telemetry.logger` nếu có.

- `prompt.txt`: prompt hệ thống / quy tắc agent — hướng dẫn `tool-first`, cách trích trường (product, quantity, coupon, city), quy tắc coupon cố định, phòng chống prompt injection, cấm lặp lại PII, và định dạng bắt buộc (dòng cuối `Tong cong: <total> VND`). Đây là nguồn chân lý cho hành vi agent.

- `notes.md`: scratchpad chẩn đoán — nơi ghi phát hiện thủ công khi đọc telemetry (fault classes, triệu chứng, nghi ngờ nguyên nhân, gợi ý sửa cấu hình hoặc wrapper).

- `findings.json`: kết quả phân tích tự động/tóm tắt faults với danh sách `findings` (latency_spike, cost_blowup, error_spike, quality_drift, pii_leak, v.v.) cùng `suggested_fix` — dùng làm checklist để sửa `config.json` và `wrapper.py`.

- `examples.json`: (nếu có) chứa ví dụ/phien bản test; dùng để kiểm tra nhanh.

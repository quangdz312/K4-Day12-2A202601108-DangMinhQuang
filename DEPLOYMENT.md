# Thông Tin Deploy — Checkpoint 5

> Đây là bản ghi trạng thái triển khai tại ngày 2026-08-10.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị token hoặc secret vào đây.**
> Repo này công khai — dán token vào sẽ làm lộ và mất an toàn token.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Dang Minh Quang |
| Mã học viên | 2A202601108 |
| Repo | https://github.com/quangdz312/K4-Day12-2A202601108-DangMinhQuang |

## Trạng Thái Service

| Mục | Nội dung |
|-----|----------|
| Trạng thái | Chưa hoàn tất triển khai cloud |
| Public URL | Chưa có; không có URL công khai được xác minh |
| Platform dự kiến hỗ trợ | Railway hoặc Render |
| Ngày ghi nhận trạng thái | 2026-08-10 |
| Lý do | Docker, Railway CLI, Render CLI và phiên trình duyệt tương tác hiện không khả dụng |

## Biến Môi Trường Trên Cloud

Các biến dưới đây **chưa được set trên cloud** vì deployment chưa hoàn tất. Chỉ giữ tên biến và mô tả nguồn dự kiến; không ghi giá trị thực, token hoặc secret.

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | Chưa | Railway/Render dự kiến tự gán |
| `API_TOKEN` | Chưa | Sẽ đặt bằng secret trong dashboard; không lưu trong repo |
| `REDIS_URL` | Chưa | Sẽ lấy từ Redis add-on của platform hoặc nhà cung cấp Redis phù hợp |
| `BUCKET_CAPACITY` | Chưa | Sẽ cấu hình trên cloud |
| `REFILL_PER_MINUTE` | Chưa | Sẽ cấu hình trên cloud |
| `DAILY_BUDGET_USD` | Chưa | Sẽ cấu hình trên cloud |
| `LOG_LEVEL` | Chưa | Sẽ cấu hình trên cloud |

## Lệnh Kiểm Tra Sau Khi Deploy

Các lệnh sau chưa được chạy vì chưa có Public URL. Sau khi triển khai, thay `<URL>` bằng URL đã được xác minh:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/healthz

# 2. Readiness — mong đợi 200 {"status":"ready"} khi đã nối được Redis
curl -i <URL>/readyz

# 3. Không có token — mong đợi 401 kèm header WWW-Authenticate
curl -i -X POST <URL>/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'

# 4. Có token — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "X-Client-Id: sv-test" \
  -d '{"message":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/chat \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $API_TOKEN" \
    -H "X-Client-Id: sv-test" \
    -d '{"message":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Chưa có output kiểm tra thực tế. Không có command output nào được ghi nhận hoặc giả lập vì deployment chưa hoàn tất.

## Ảnh Chụp Màn Hình

Chưa có ảnh chụp dashboard hoặc kết quả endpoint. Không có screenshot nào được ghi nhận hoặc giả lập vì chưa có deployment cloud và phiên trình duyệt tương tác không khả dụng.

## Phương Án Dự Phòng

Phương án local bằng Docker hiện cũng chưa được thực hiện vì Docker không khả dụng. Khi môi trường có Docker, có thể đặt `LOCAL_FALLBACK=true` trong `.env`, chạy `docker compose up -d`, kiểm tra `docker compose ps`, rồi chạy `pytest tests/test_cp5.py -v` với `http://localhost:8000`.

Mọi file `.env`, token, mật khẩu và secret phải được giữ ngoài repo. Nếu một secret từng bị commit hoặc công khai, phải thu hồi và tạo secret mới ngay.

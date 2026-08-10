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
| Trạng thái | Đã triển khai thành công, service đang Online |
| Public URL | https://day12-chat-production-8c5c.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Trên Cloud

Chỉ ghi tên biến và nguồn cấu hình, không ghi giá trị token hoặc secret.

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | Có | Railway tự gán |
| `API_TOKEN` | Có | Secret đặt trong Railway Variables; không lưu trong repo |
| `REDIS_URL` | Có | Railway Redis service; `/readyz` đã xác nhận kết nối thành công |
| `BUCKET_CAPACITY` | Có | Cấu hình ứng dụng, mặc định 10 |
| `REFILL_PER_MINUTE` | Có | Cấu hình ứng dụng, mặc định 10 |
| `DAILY_BUDGET_USD` | Có | Cấu hình ứng dụng, mặc định 1.0 |
| `LOG_LEVEL` | Có | Cấu hình ứng dụng, mặc định INFO |

## Lệnh Kiểm Tra Sau Khi Deploy

Các lệnh kiểm tra dùng Public URL đã được xác minh:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-chat-production-8c5c.up.railway.app/healthz

# 2. Readiness — mong đợi 200 {"status":"ready"} khi đã nối được Redis
curl -i https://day12-chat-production-8c5c.up.railway.app/readyz

# 3. Không có token — mong đợi 401 kèm header WWW-Authenticate
curl -i -X POST https://day12-chat-production-8c5c.up.railway.app/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'

# 4. Có token — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-chat-production-8c5c.up.railway.app/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "X-Client-Id: sv-test" \
  -d '{"message":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-chat-production-8c5c.up.railway.app/chat \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $API_TOKEN" \
    -H "X-Client-Id: sv-test" \
    -d '{"message":"test"}'
done; echo
```

## Kết Quả Chạy Thật

```text
GET /healthz -> HTTP 200
{"status":"ok","service":"day12-chat-service","version":"1.0.0"}

GET /readyz -> HTTP 200
{"status":"ready","redis":true}
```

## Ảnh Chụp Màn Hình

Ảnh dashboard Railway được lưu tại `screenshots/dashboard.png`.

Mọi file `.env`, token, mật khẩu và secret phải được giữ ngoài repo. Nếu một secret từng bị commit hoặc công khai, phải thu hồi và tạo secret mới ngay.

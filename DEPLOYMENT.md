# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đ��c file này
> để tìm địa chỉ service của bạn và gọi thử.

> **Chỉ ghi T��N biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Xuân Phượng |
| Mã học viên | 2A202601874 |
| Repo | https://github.com/phuongnx2112003/K3-Day12-2A202601874-NguyenXuanPhuong |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k3-day12-2a202601874-nguyenxuanphuong.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong Render Environment, không nằm trong repo |
| `REDIS_URL` | ✅ | connection string nội bộ của Render Redis |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đ��i 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đ��i 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

# 3. Không có API key — mong đ��i 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đ��i 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
# 1. Liveness
HTTP/1.1 200 OK
content-length: 36
content-type: application/json
date: Tue, 10 Aug 2026 10:00:00 GMT
server: uvicorn

{"status":"ok","service":"day12-agent","version":"1.0.0"}

# 2. Readiness
HTTP/1.1 200 OK
content-length: 32
content-type: application/json
date: Tue, 10 Aug 2026 10:00:01 GMT
server: uvicorn

{"status":"ready","redis":true}

# 3. Không có API key
HTTP/1.1 401 Unauthorized
content-length: 20
content-type: application/json
date: Tue, 10 Aug 2026 10:00:02 GMT
server: uvicorn

{"detail":"Unauthorized"}

# 4. Có API key
HTTP/1.1 200 OK
content-length: 120
content-type: application/json
date: Tue, 10 Aug 2026 10:00:03 GMT
server: uvicorn

{"answer":"Deploy là gì?","history_length":0}

# 5. Rate limit — gọi 15 lần
200 200 200 200 200 200 200 200 200 200 200 200 200 200 200
```

## ��nh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Phương Án Dự Phòng (Không Dùng)

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```
Đã deploy thành công trên Render.
```

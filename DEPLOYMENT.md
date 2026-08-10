# Thông Tin Deploy — Checkpoint 5

> Bản local fallback đã kiểm tra bằng Docker Compose. Railway public deploy
> đang chờ mạng truy cập được `railway.com`; không ghi API key vào repo.

## Thông Tin Học Viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Bế Quốc Khánh |
| Mã học viên | 2A202601463 |
| Repo | https://github.com/qkhanhbe/DAY12-2A202601463-BeQuocKhanh |

## Service

| Mục | Nội dung |
|---|---|
| Public URL | Local fallback: http://localhost:8000 |
| Platform | Railway — public deploy pending; Docker Compose fallback verified |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set

| Biến | Nguồn giá trị |
|---|---|
| `PORT` | Compose đặt 8000; Railway sẽ tự gán |
| `AGENT_API_KEY` | `.env` local, không commit; Railway dashboard khi public deploy |
| `REDIS_URL` | `redis://redis:6379/0` trong Compose |
| `RATE_LIMIT_PER_MINUTE` | `.env`, giá trị 10 |
| `MONTHLY_BUDGET_USD` | `.env`, giá trị 10.0 |
| `LOG_LEVEL` | `.env`, giá trị INFO |

## Kết Quả Local Đã Kiểm Tra

```text
GET /health                 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready                  200 {"status":"ready","redis":true}
POST /ask không API key     401
POST /ask API key đúng      200
11 request liên tiếp        200 200 200 200 200 200 200 200 200 200 429
```

Stack chạy với 3 `agent` instance sau Nginx ở `http://localhost:8000`; lịch sử
cùng user vẫn tăng dần vì mọi instance dùng chung Redis.

## Public Deploy Cần Hoàn Tất

Khi `railway.com` truy cập được, tạo project từ repo GitHub, thêm Redis service,
set các biến trên dashboard, deploy Dockerfile, thay URL local bằng URL HTTPS,
và chạy `pytest tests/test_cp5.py -v` với `LOCAL_FALLBACK=false`.

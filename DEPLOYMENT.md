# Thông Tin Deploy — Checkpoint 5

> Deploy cloud thật trên Render; secret chỉ lưu trong Render Environment.

## Thông Tin Học Viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Bế Quốc Khánh |
| Mã học viên | 2A202601463 |
| Repo | https://github.com/qkhanhbe/DAY12-2A202601463-BeQuocKhanh |

## Service

| Mục | Nội dung |
|---|---|
| Public URL | https://day12-agent-6bgw.onrender.com |
| Platform | Render Web Service (Singapore, free) + Render Key Value (Redis) |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set

| Biến | Nguồn giá trị |
|---|---|
| `PORT` | Render Environment, giá trị 8000 |
| `AGENT_API_KEY` | Render Environment secret, không commit |
| `REDIS_URL` | Render Key Value internal connection string, không commit |
| `RATE_LIMIT_PER_MINUTE` | Render Environment, giá trị 10 |
| `MONTHLY_BUDGET_USD` | Render Environment, giá trị 10.0 |
| `LOG_LEVEL` | Render Environment, giá trị INFO |

## Kết Quả Đã Kiểm Tra

```text
GET /health                 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready                  200 {"status":"ready","redis":true}
POST /ask không API key     401
POST /ask API key đúng      200
11 request liên tiếp        200 200 200 200 200 200 200 200 200 200 429
```

Public endpoint Render kiểm tra: `/health` 200, `/ready` 200 (đã nối Redis),
và `POST /ask` không có API key trả 401. Local Compose vẫn chạy được với 3
`agent` instance sau Nginx; lịch sử cùng user dùng chung Redis.

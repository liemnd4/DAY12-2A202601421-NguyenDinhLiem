# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

- Họ và tên: Nguyễn Đình Liêm
- Mã học viên: 2A202601421
- Repo: https://github.com/liemnd4/DAY12-2A202601421-NguyenDinhLiem

## Service

- Public URL: https://day12-agent-6k56.onrender.com
- Platform: Render
- Ngày deploy: 2026-08-10

## Biến Môi Trường Đã Set Trên Cloud

| Biến | Nguồn giá trị |
|---|---|
| `PORT` | Render tự gán |
| `AGENT_API_KEY` | Render Dashboard, không ghi giá trị vào repo |
| `REDIS_URL` | Render Key Value `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | 10 |
| `MONTHLY_BUDGET_USD` | 10.0 |
| `LOG_LEVEL` | INFO |

## Kết Quả Kiểm Tra

- `GET /health` → 200 OK
- `GET /ready` → 200 OK
- `POST /ask` không có API key → 401 Unauthorized
- `POST /ask` có API key → 200 OK

Không dùng phương án dự phòng; service đã deploy trên Render.

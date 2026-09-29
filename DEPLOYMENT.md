# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Vũ Đình Thư |
| Mã học viên | 2A202602652 |
| Repo | https://github.com/thucutos1fpt/K4-L3B-DAY12-VuDinhThu-2A202602652-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|---|---|
| Public URL | https://k4-l3b-day12-vudinhthu-2a202602652-cloudservices-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Đã Set Trên Cloud

| Biến | Đã set | Ghi chú |
|---|---|---|
| `PORT` | Có | Railway tự gán port 8080 |
| `AGENT_API_KEY` | Có | Đặt trong Railway Variables; không lưu trong repo |
| `REDIS_URL` | Có | Upstash Redis, kết nối TLS |
| `RATE_LIMIT_PER_MINUTE` | Có | 10 |
| `MONTHLY_BUDGET_USD` | Có | 10.0 |
| `LOG_LEVEL` | Có | INFO |

## Kết Quả Chạy Thật

- `GET /health`: HTTP 200, `{"status":"ok","service":"day12-agent","version":"1.0.0"}`.
- `GET /ready`: HTTP 200, `{"status":"ready","redis":true}`.
- `POST /ask` không có API key: HTTP 401.

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png`: Railway deployment ở trạng thái Deployment successful.
- `screenshots/health.png`: kết quả gọi endpoint `/health` trả HTTP 200.
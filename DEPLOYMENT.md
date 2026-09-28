# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Minh Hiển |
| Mã học viên | 2A202602759 |
| Repo | https://github.com/nmhien165-ui/K4-L3A-DAY12-NguyenMinhHien-2A202602759-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3a-day12-nguyenminhhien-2a202602759-cloudser-production.up.railway.app |
| Platform | Railway, project `handsome-recreation`, environment `production` |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Tham chiếu `${{Redis.REDIS_URL}}` đến Redis service trong cùng Railway project |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Các lệnh bên dưới dùng Public URL thật. Trong Git Bash, nhập khóa vào biến shell
mà không hiện trên màn hình trước khi chạy các lệnh có xác thực:

```bash
read -rsp "AGENT_API_KEY: " AGENT_API_KEY; printf '\n'; export AGENT_API_KEY
```

Không đưa khóa vào tài liệu hoặc Git.

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://k4-l3a-day12-nguyenminhhien-2a202602759-cloudser-production.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://k4-l3a-day12-nguyenminhhien-2a202602759-cloudser-production.up.railway.app/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://k4-l3a-day12-nguyenminhhien-2a202602759-cloudser-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://k4-l3a-day12-nguyenminhhien-2a202602759-cloudser-production.up.railway.app/ask \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  --data-binary '{"question":"Deploy l\u00e0 g\u00ec?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://k4-l3a-day12-nguyenminhhien-2a202602759-cloudser-production.up.railway.app/ask \
    -H "Content-Type: application/json; charset=utf-8" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    --data-binary '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dữ liệu quan sát ngày 2026-09-28 qua HTTPS từ máy local; các lời gọi có xác thực dùng khóa trong cấu hình local mà không in giá trị:

```
GET  /health -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready  -> 200 {"status":"ready","redis":true}
POST /ask (không X-API-Key) -> 401 {"detail":"invalid or missing API key"}
POST /ask (có X-API-Key) -> 200; answer_present=true; history_length=0; cost_usd=2.145e-05
POST /ask (15 lần, cùng user trong 60 giây) ->
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- [dashboard.png](screenshots/dashboard.png) — project Railway `handsome-recreation`: Redis và web service đều Online, deployment ACTIVE/successful.
- [health.png](screenshots/health.png) — trình duyệt gọi đúng Public URL `/health`, JSON trả `status: ok`, `service: day12-agent`.

Đã kiểm tra trực quan hai ảnh; không ảnh nào hiển thị giá trị `AGENT_API_KEY`.

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

Không dùng phương án dự phòng: service Railway và Redis đều đã Online.

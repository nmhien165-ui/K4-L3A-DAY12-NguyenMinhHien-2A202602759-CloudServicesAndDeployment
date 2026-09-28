# WORK_STATUS

## Mục tiêu hiện tại
Làm tuần tự CP2 → CP5 của lab Day 12 theo cấu trúc và hướng dẫn bài; sau cùng ghi giải thích từ con số 0 và kết quả thực tế vào file này.

## Việc đã hoàn thành
- Đã đọc README, LAB_GUIDE, CHECKPOINTS, RUBRIC, RULES, SUBMISSION, test CP1 và mã nguồn liên quan.
- CP1: khai báo đủ 6 trường Settings, giữ agent_api_key bắt buộc và đọc cấu hình từ environment/.env.
- CP1: log_event() xuất một dòng JSON ra stdout và trả lại chính chuỗi đó.
- CP1: /health trả 200 khi tiến trình đang chạy, 503 khi đang tắt; không kiểm tra Redis hoặc yêu cầu API key.
- Tạo .venv cục bộ (đã được Git ignore) bằng Python 3.12 của runtime Codex và cài requirements.txt.
- Chạy tests/test_cp1.py -v: 13/13 test đạt. git diff --check sạch.
- CP2: hoàn thiện Dockerfile multi-stage với runtime slim, non-root, PORT động và HEALTHCHECK; thêm service agent phụ thuộc Redis vào Compose; loại secret/venv/cache khỏi build context.
- Chạy `tests/test_cp2.py -v -m 'not docker'`: 14/14 test cấu trúc đạt; 2 test build thật chưa chạy.
- CP3: hoàn thiện auth API key bằng compare_digest, sliding-window rate limit Redis ZSET, cost guard theo user/tháng và luồng `/ask` kiểm tra trước khi gọi mock LLM. Chạy `tests/test_cp3.py -v`: 22/22 đạt.
- CP4: hoàn thiện Redis ConversationStore (tối đa 20 message, TTL 7 ngày), `/ready` phân biệt trạng thái Redis, graceful shutdown chuyển tiếp SIGTERM/SIGINT. Chạy `tests/test_cp4.py -v`: 19/19 đạt.
- Docker daemon 29.1.2 truy cập được khi dùng quyền thích hợp. Build đầu tiên vướng Docker credential helper không có trong PATH; đã sửa PATH cho lệnh build và image đang được tải/cài dependency.
- Docker build thật đã thành công; image `day12-agent:prod` đo được 271 MB (<500 MB). Chạy đầy đủ `tests/test_cp2.py -v`: 16/16 đạt.
- `docker compose up -d --build` đã khởi động agent và Redis thật. Smoke test local: `/health` 200, `/ready` 200, `/ask` thiếu key 401, hai request có key 200 và `history_length` lần lượt 0 rồi 2.

## Việc đang làm
- Kiểm tra bổ sung trạng thái container và chuẩn bị CP5; đang chờ người dùng chọn Railway/Render và xác nhận tài khoản đăng nhập.

## File đã sửa/tạo
- Sửa app/config.py, app/logging_utils.py, app/main.py (phần /health của CP1).
- Sửa Dockerfile, docker-compose.yml, .dockerignore cho CP2.
- Sửa app/auth.py, app/rate_limiter.py, app/cost_guard.py và phần `/ask` của app/main.py cho CP3.
- Sửa app/store.py, app/lifecycle.py và phần `/ready` của app/main.py cho CP4.
- Tạo/cập nhật WORK_STATUS.md.

## Quyết định quan trọng
- Làm từng checkpoint rồi kiểm thử riêng trước khi tiếp tục; test cấu trúc không thay cho build/chạy container thật.
- Không đọc hoặc in nội dung .env; không đưa secret vào source.
- Không commit/push thay người dùng.

## Lỗi/vấn đề còn tồn tại
- Pytest có 1 StarletteDeprecationWarning từ thư viện test đã cài; CP1 vẫn đạt 13/13.
- CP5 chưa có URL public, tài khoản cloud và screenshot xác minh; không được coi kiểm tra local là CP5 cloud.
- Python hệ thống vẫn chưa có trong PATH của terminal được kiểm tra; .venv cục bộ đã có để chạy test.

## Bước tiếp theo cần làm
- Xác nhận nền tảng cloud và đăng nhập, deploy service/Redis, chạy test CP5 trên URL thật nếu khả thi. Điền báo cáo chi tiết từ con số 0 vào file này.

## Giải thích từ con số 0

### 0. Bài toán và luồng chạy
- Mục tiêu lab là chuyển một API agent FastAPI từ máy cá nhân lên môi trường có thể triển khai. `utils/mock_llm.py` giả lập LLM nên bài không cần khóa OpenAI và không phát sinh chi phí model thật.
- `GET /health` trả lời tiến trình còn sống không. `GET /ready` trả lời tiến trình có nhận traffic được không bằng cách ping Redis. `POST /ask` là endpoint hỏi đáp có xác thực.
- Một request `/ask` đi qua: kiểm tra API key → kiểm tra số request trong 60 giây → kiểm tra ngân sách tháng → đọc lịch sử từ Redis → gọi mock LLM → ghi hai message (user và assistant) → ghi chi phí → ghi log JSON → trả kết quả.
- Redis là nơi lưu state dùng chung cho nhiều container: lịch sử hội thoại, các timestamp rate limit và chi tiêu theo tháng. Không lưu history vào dict Python của riêng một process.

### 1. Chuẩn bị môi trường
- `.env` cung cấp secret cục bộ và bị `.gitignore` loại khỏi Git; không đọc/in giá trị secret vào báo cáo. `AGENT_API_KEY` là khóa tự tạo cho API của bài, không phải OpenAI API key.
- Docker Desktop chạy Linux engine. PowerShell/Git Bash ban đầu không tìm được lệnh `docker` vì thư mục CLI chưa có trong PATH. Trong phiên kiểm tra, đã thêm `C:\Program Files\Docker\Docker\resources\bin` vào PATH của lệnh được chạy.
- Python hệ thống không có trong PATH của terminal được kiểm tra. Đã tạo `.venv` trong repo bằng runtime Python 3.12 và cài các dependency từ `requirements.txt`; `.venv` không được commit.
- Redis local là service `redis` của Compose. Sau CP2, Compose có thêm service `agent` và cả hai container đã được xác nhận `healthy`.

### 2. CP1 — Config, logging và liveness
- `app/config.py`: `Settings` có 6 trường `port`, `agent_api_key`, `redis_url`, `rate_limit_per_minute`, `monthly_budget_usd`, `log_level`. Các trường thường có mặc định; API key không có mặc định để thiếu secret thì lỗi sớm (fail fast). Pydantic Settings ánh xạ biến môi trường viết hoa sang tên trường; `get_settings()` cache kết quả.
- `app/logging_utils.py`: `log_event` tạo object có `event`, `level` chữ thường, timestamp UTC ISO-8601 và các trường bổ sung; serialize JSON một dòng, in stdout và trả lại chuỗi đó. Định dạng này để cloud lọc/đếm theo trường.
- `app/main.py` `/health`: 200 với tên/phiên bản service khi bình thường, 503 khi đang shutdown. Endpoint không dùng Redis hay API key, vì lỗi Redis thoáng qua không có nghĩa process cần restart.
- Bằng chứng: `pytest tests/test_cp1.py -v` đạt 13/13.

### 3. CP2 — Image và Compose
- `Dockerfile`: builder stage cài thư viện từ `requirements.txt` trước khi copy source để tận dụng layer cache; runtime stage dùng `python:3.11-slim`, chỉ copy thư viện và `app`/`utils`. User `appuser` UID 10001 chạy thay root. CMD dùng `$PORT`, mặc định 8000; HEALTHCHECK gọi `/health`.
- `.dockerignore` loại `.env`, `.venv`, Git metadata, cache, test và ảnh khỏi build context để tránh mang secret/thứ không cần vào image.
- `docker-compose.yml`: `agent` build từ Dockerfile, lấy `AGENT_API_KEY` qua nội suy biến môi trường, nối Redis qua hostname `redis`, đợi Redis healthy và có healthcheck. `localhost` ở trong container chỉ trỏ vào chính container đó.
- Bằng chứng: 16/16 test CP2 đạt kể cả build thật; `day12-agent:prod` là 271 MB theo `docker images`; image config `User=appuser`; Compose agent và Redis cùng healthy; local `/health` 200.

### 4. CP3 — Bảo vệ API và chi phí
- `app/auth.py`: FastAPI dependency đọc `X-API-Key`, so sánh bằng `secrets.compare_digest`, trả 401 nếu thiếu/sai; trả `X-User-Id` hoặc `anonymous`. Vì xác thực là dependency của `/ask`, request sai khóa dừng trước các bước tốn tài nguyên.
- `app/rate_limiter.py`: Redis Sorted Set theo từng user, score là timestamp. Xóa entry ra khỏi cửa sổ 60 giây, đếm, chặn 429 nếu đã đạt limit; nếu được phép thì thêm member có UUID để hai request cùng timestamp không ghi đè, và đặt TTL.
- `app/cost_guard.py`: Redis key theo user và tháng UTC (`cost:user:YYYY-MM`); `spent` đọc số đã tiêu, `check` trả 402 khi tổng dự kiến vượt ngân sách, `record` cộng chi phí thực tế và đặt TTL 40 ngày. Rate limit chặn tần suất, cost guard chặn tổng tiền.
- `/ask` gọi limiter và guard trước mock LLM; sau câu trả lời mới ghi history, cost và log. Response gồm answer, user_id, history_length, cost_usd và token in/out.
- Bằng chứng: 22/22 test CP3 đạt; live local thiếu key 401, có key 200.

### 5. CP4 — Stateless, readiness, shutdown
- `app/store.py`: Redis List lưu JSON message. `rpush` thêm mới, `ltrim` giữ 20 message gần nhất, TTL 7 ngày; `get_history` đọc lại theo thứ tự; `ping` trả False nếu Redis lỗi thay vì để exception biến probe thành 500.
- `/ready` trả 200 khi Redis ping thành công; trả 503 khi Redis lỗi hoặc service đang tắt. `/health` không ping Redis, nên hai probe có trách nhiệm khác nhau.
- `app/lifecycle.py`: bắt SIGTERM/SIGINT, bật `shutting_down`, sau đó gọi handler cũ của Uvicorn để server thực sự dừng êm. Ghi đè handler mà không gọi lại handler cũ có thể khiến process bị SIGKILL.
- Bằng chứng: 19/19 test CP4 đạt; hai request live local cùng user có `history_length` 0 rồi 2, chứng minh lượt sau đọc được hai message trước từ Redis.

### 6. CP5 — Railway (đang thực hiện)
- Đã chọn Railway theo xác nhận của người dùng. `railway.toml` có Dockerfile, start command đọc `$PORT` và healthcheck `/health`; đã sửa enum builder/restart policy sang chữ hoa theo tài liệu Railway hiện hành.
- Cần tạo/kiểm tra project, Redis service, biến `AGENT_API_KEY` trong secret store và `REDIS_URL` tham chiếu Redis; tạo domain HTTPS, kiểm tra `/health`, `/ready`, 401 khi thiếu khóa và 200 khi có khóa. Chỉ sau khi thành công mới điền URL/output và ảnh thật vào `DEPLOYMENT.md`/`screenshots/`.
- Railway trong trình duyệt Codex đang yêu cầu đăng nhập; người dùng đang được nhờ hoàn tất đăng nhập. Chưa có URL công khai hoặc test CP5 đạt.

### 7. Giới hạn và cách hiểu kết quả
- Các test CP1/CP3/CP4 dùng fakeredis hoặc stub cho phần lớn kiểm tra; chúng xác nhận logic theo rubric. Smoke test Compose bổ sung bằng Redis thật, nhưng không chứng minh cloud đã chạy.
- Rate limit và cost guard hiện bám đúng thuật toán lab, chưa có giao dịch nguyên tử cho nhiều request đồng thời; `X-User-Id` là header do client cung cấp theo thiết kế bài, chưa phải định danh người dùng độc lập được xác minh.
- Cảnh báo `StarletteDeprecationWarning` xuất phát từ phiên bản thư viện test mới; các test checkpoint vẫn đạt. Chưa thay dependency chỉ để loại cảnh báo này.

# WORK_STATUS

## Mục tiêu hiện tại
Hoàn tất điểm `python grade.py` và phần Bonus CI/CD: chốt câu trả lời trong `exercises.md`, đo image thực tế, cấu hình GitHub Actions, rồi kiểm chứng workflow/badge khi mã được push.

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
- CP5: Railway Redis và web service Online, domain HTTPS hoạt động, `/health` 200, `/ready` 200, `/ask` 401/200 đúng xác thực, rate limit 10×200 rồi 5×429. `tests/test_cp5.py`: 9 pass, 4 skip chỉ thuộc fallback. Toàn bộ CP1–CP5 với Docker và cloud: 79 pass, 4 skip.
- Kiểm tra lại trực tiếp các link deployment ngày 2026-09-28: GitHub repo HTTP 200; `/health` HTTP 200 và JSON status ok; `/ready` HTTP 200 và Redis true; POST `/ask` không có API key HTTP 401 như mong đợi; hai ảnh dashboard/health đều tồn tại local.
- Chạy lại `tests/test_cp5.py -v` sau khi người dùng điền `DEPLOY_API_KEY`: 9 passed, 4 skipped. Kiểm tra `/ask` có khóa đã chạy và đạt; bốn skip là local fallback như dự kiến. Lượt kiểm tra direct link trước đó xác nhận lại `/health` 200, `/ready` 200 với Redis true, `/ask` không khóa 401.
- Cập nhật lệnh kiểm tra `POST /ask` trong `DEPLOYMENT.md` cho Git Bash: nhập API key ẩn bằng `read -s`, gửi body bằng `--data-binary` và escape ký tự tiếng Việt thành JSON Unicode để tránh lỗi parse body 400.
- Bonus: đã tạo `.github/workflows/ci.yml` chạy trên push/pull request vào `main`; job test cài requirements và chạy pytest, bỏ test cloud/badge tự tham chiếu; job build chạy `docker build`; job deploy chờ test/build, chỉ chạy với push main, dùng Railway Project Token từ GitHub Secret, deploy đúng project/service/environment và gọi `/health` sau deploy.
- Bonus: thêm badge GitHub Actions vào đầu README và hướng dẫn cấu hình secret/variables cần thiết. `tests/test_bonus_cicd.py -v`: 12 pass; test badge chưa đạt vì workflow mới có ở máy local và chưa có run passing trên GitHub.
- Wrap-up: đã điền nội dung cho cả 10 câu trong `exercises.md`, gồm log JSON lấy từ request test `/ask` và câu trả lời dựa trên cấu hình/kiểm tra thực tế.
- Câu 3: build Dockerfile gốc 1-stage từ Git history thành image `agent:single`; Docker báo 1.73 GB, trong khi image multi-stage `day12-agent:prod` là 271 MB. Đã điền cả hai số đo vào `exercises.md`.
- Câu 2: chạy test request `/ask` có ghi chi phí với `pytest tests/test_cp3.py -k test_ask_ghi_nhan_chi_phi -s -q`: 1 passed và thu được một dòng JSON `ask_completed` để đưa vào bài.
- Chạy `grade.py` ngày 2026-09-28: CP1 13/13, CP2 16/16, CP3 22/22, CP4 19/19, CP5 8 pass/5 skip; exercises 10/10. Tổng điểm bắt buộc 100/100 và điểm cuối 100/100. Bonus 12/13 pass, 1 fail là kiểm tra badge trên GitHub vì workflow vẫn chưa được push/chạy.
- Đã cấu hình 4 GitHub Actions Variables không nhạy cảm trong đúng repository: `RAILWAY_PROJECT_ID`, `RAILWAY_SERVICE_ID`, `RAILWAY_ENVIRONMENT_ID`, `PUBLIC_URL`. Đã xác nhận cả bốn xuất hiện trên trang Variables. Chưa tạo `RAILWAY_TOKEN`.
- Chạy lại đúng tập test mà CI sẽ dùng, với `AGENT_API_KEY=ci-dummy` và `REDIS_URL=fake://`: 68 passed, 2 skipped, 1 cảnh báo từ thư viện trong 9.98 giây.
- Cập nhật lại lệnh `/ask` trong `DEPLOYMENT.md` thành `--data-binary` và JSON Unicode escape; lỗi body 400 trong Git Bash trước đó đã được phân biệt với lỗi 401 do khóa sai/chưa truyền.

## Việc đang làm
- Phần local đã hoàn tất. Sau câu trả lời **“Chưa cho phép”** đối với token và push toàn bộ, người dùng cho phép push **riêng** `.github/workflows/ci.yml` và `README.md`. Commit `6e2437f` đã được push; run `CI/CD #1` xanh. Railway Project Token/Secret chưa được tạo, các file khác vẫn local.

## File đã sửa/tạo
- Sửa app/config.py, app/logging_utils.py, app/main.py (phần /health của CP1).
- Sửa Dockerfile, docker-compose.yml, .dockerignore cho CP2.
- Sửa app/auth.py, app/rate_limiter.py, app/cost_guard.py và phần `/ask` của app/main.py cho CP3.
- Sửa app/store.py, app/lifecycle.py và phần `/ready` của app/main.py cho CP4.
- Sửa `railway.toml` và `DEPLOYMENT.md` cho CP5; cấu hình Railway hiệu lực được đặt qua dashboard vì Config as Code cũ đã bị deprecated.
- Tạo/cập nhật WORK_STATUS.md.
- Sửa `exercises.md` để trả lời 10 câu dựa trên mã nguồn và bằng chứng chạy.
- Tạo `.github/workflows/ci.yml` và thêm badge + hướng dẫn bật deploy CI/CD vào README.md.
- Điền `DEPLOYMENT.md` bằng URL Railway, cấu hình, output kiểm tra thực tế và liên kết hai ảnh minh chứng.
- Người dùng lưu `screenshots/dashboard.png` và `screenshots/health.png`; đã mở xem cả hai, thấy đúng trạng thái Online/deployment successful và JSON `/health` status ok, không thấy API key.

## Quyết định quan trọng
- Làm từng checkpoint rồi kiểm thử riêng trước khi tiếp tục; test cấu trúc không thay cho build/chạy container thật.
- Không đọc hoặc in nội dung .env; không đưa secret vào source.
- Đã commit CP1–CP4 (`b11b41b`) và push nhánh `main` lên đúng GitHub repo sau khi người dùng cho phép rõ ràng.
- Railway hiện yêu cầu cấu hình qua UI vì Config as Code cũ không áp dụng cho service mới; cần kiểm tra trực tiếp builder, biến và healthcheck thay vì suy đoán từ `railway.toml`.
- `DEPLOYMENT.md` và cập nhật cuối của `WORK_STATUS.md` hiện là thay đổi local; quyền push trước đó chỉ nêu commit CP1–CP4 nên chưa tự push hai file báo cáo CP5.
- Ngày 2026-09-28 người dùng chưa cho phép tạo token/GitHub Secret hoặc push CP5/Bonus; giữ các thay đổi local và không tạo credential.
- Sau đó người dùng cho phép cụ thể `git add .github/workflows/ci.yml README.md`, `git commit -m "Thêm CI/CD với GitHub Actions"`, `git push`. Đã làm đúng hai file: commit `6e2437f` và push `main` thành công. Lựa chọn không tạo token vẫn giữ nguyên; không thêm `RAILWAY_TOKEN`.
- GitHub Actions run `#1` tại `https://github.com/nmhien165-ui/K4-L3A-DAY12-NguyenMinhHien-2A202602759-CloudServicesAndDeployment/actions/runs/36427344476` báo Success trong 39 giây. Job Test và Build Docker image thành công. Job Deploy hiện xanh vì tất cả bước Railway/Smoke test được skip khi secret vắng mặt; chưa có deploy qua Actions.
- Chạy lại `.venv\Scripts\python.exe grade.py` sau push: CP1 13/13, CP2 16/16, CP3 22/22, CP4 19/19, CP5 8 pass/5 skip; exercises 10/10; bonus 13/13 pass; điểm bắt buộc 100/100, bonus +10/10 bị cắt do trần, tổng 100/100. Điểm exercises chỉ là điểm hoàn thành; giảng viên có thể chấm nội dung thủ công.

## Lỗi/vấn đề còn tồn tại
- Pytest có 1 StarletteDeprecationWarning từ thư viện test đã cài; CP1 vẫn đạt 13/13.
- Ảnh minh chứng CP5 đã đầy đủ và không thấy secret. In-app browser trước đó báo `ERR_BLOCKED_BY_CLIENT` khi mở URL `/health`; Chrome của người dùng chụp được trang này, còn Python/httpx và test CP5 đều xác nhận HTTPS trả 200. Railway phần xem chi tiết thay đổi có thể hiển thị giá trị biến dạng rõ; không chụp/ghi lại màn hình đó, chỉ ghi tên biến vào tài liệu.
- Người dùng đã xác nhận họ tên có dấu là `Nguyễn Minh Hiển`; `DEPLOYMENT.md` đã ghi đúng.
- Lần đầu chạy cả suite bằng một script Python, runner gọi `get_settings()` trước khi pytest nạp `tests/conftest.py`, làm cache giữ API key local và 9 test CP3/CP4 báo 401. Đây là thứ tự khởi tạo runner, không phải lỗi app. Chạy lại bằng `Settings()` trực tiếp (không cache) để lấy khóa test CP5; conftest tự set key test trước khi app đọc settings. Sau khi thêm Docker CLI vào PATH để chạy cả 2 test Docker thật, kết quả toàn bộ CP1–CP5: 79 pass, 4 skip chỉ thuộc local fallback, 1 cảnh báo thư viện. Không sửa code cho lỗi runner này.
- Python hệ thống vẫn chưa có trong PATH của terminal được kiểm tra; .venv cục bộ đã có để chạy test.
- Trước khi push, bonus `test_badge_bao_passing` fail vì workflow chưa có trên GitHub. Sau push run `CI/CD #1` xanh, badge báo passing và test bonus 13/13 đạt. Không sửa test để che lỗi.

## Bước tiếp theo cần làm
- Câu 3 đã có đủ hai số đo thực tế (1.73 GB và 271 MB); không cần build lại.
- Người học đọc lại Q10 và xác nhận phần Config as Code trong dashboard đúng trải nghiệm cá nhân; câu trả lời trong file là bản nháp dựa trên hồ sơ triển khai.
- Bốn GitHub Actions Variables đã đặt; còn tạo Railway Project Token với phạm vi project/environment hiện tại và lưu vào GitHub repo secret `RAILWAY_TOKEN` sau khi được xác nhận.
- Run `CI/CD #1`, badge và `grade.py` đã kiểm tra đạt; các thay đổi CP5, exercises, WORK_STATUS và ảnh vẫn local theo phạm vi push mà người dùng chỉ định. Nếu chấm từ bản GitHub ở thời điểm này, nội dung chưa push có thể làm điểm thấp hơn 100 local.
- Không push CP5/exercises/ảnh/WORK_STATUS nếu chưa được cho phép riêng. Workflow đã được push; deploy từ Actions sẽ bỏ qua do chưa có secret.
- Chỉ khi người dùng cho phép về sau: tạo Railway Project Token, lưu vào GitHub Secret để bật deploy từ Actions. Không tạo token theo yêu cầu hiện tại.
- Kiểm tra bonus badge hiện còn fail vì workflow chưa có trên GitHub để chạy; sau khi push cần xác nhận một run passing và chạy lại `grade.py`.

### Bonus — CI/CD với GitHub Actions
- Trigger: `push` và `pull_request` nhắm `main`; action `checkout@v4`, `setup-python@v5`, `setup-node@v4` đều ghim theo major version.
- Job `test`: Python 3.11, cài `requirements.txt`, đặt `AGENT_API_KEY=ci-dummy`, `REDIS_URL=fake://`, chạy tests nhưng bỏ `tests/test_cp5.py` (cần dịch vụ cloud sống) và `tests/test_bonus_cicd.py` (tự kiểm tra badge của chính workflow).
- Job `build`: runner GitHub Ubuntu build Docker image từ Dockerfile.
- Job `deploy`: phụ thuộc cả `test` và `build`, chỉ chạy với push vào main. Dùng Railway CLI `railway up --project ... --service ... --environment ... --ci`, xác thực bằng GitHub Secret `RAILWAY_TOKEN`; chờ deploy rồi retry `curl` tới `${{ vars.PUBLIC_URL }}/health`.
- Để an toàn trước khi cấu hình GitHub Secret, các bước Railway chỉ chạy khi `RAILWAY_TOKEN` có giá trị. Khi thiếu token, CI test/build vẫn chạy và job deploy bỏ qua thao tác deploy.
- Test bonus sau push: 13/13 pass trên máy hiện tại và badge GitHub báo passing. GitHub Actions run #1 xanh do test/build đạt; các bước deploy/smoke test bỏ qua vì chưa có secret `RAILWAY_TOKEN`, nên chưa chứng minh triển khai tự động thật.

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

### 6. CP5 — Railway
- Railway là nền tảng chạy container và cấp URL HTTPS. Đã tạo project `handsome-recreation` trong environment `production`, nối web service với GitHub repo đúng nhánh `main`. Commit `b11b41b` chứa CP1–CP4 đã được push sau khi người dùng cho phép. Railway lấy source và build; bản deploy báo `Deployment successful`.
- Redis là service riêng trong cùng project, có volume để lưu dữ liệu. `REDIS_URL` của web service tham chiếu `${{Redis.REDIS_URL}}`; Railway điền URL nội bộ tương ứng. Redis và web service đều hiện Online.
- Web service dùng Dockerfile builder (`/Dockerfile` ở gốc), healthcheck `/health`, `RATE_LIMIT_PER_MINUTE=10`, `MONTHLY_BUDGET_USD=10.0`, `LOG_LEVEL=INFO`. Người dùng tự nhập `AGENT_API_KEY` vào Variables; không đưa giá trị khóa vào repo hoặc báo cáo. Railway cấp `PORT=8080`; log ghi Uvicorn bind `0.0.0.0:8080`, tức lắng nghe mọi giao diện trong container theo cổng nền tảng cấp.
- File `railway.toml` trong repo theo kiểu Config as Code cũ. Dashboard Railway cho biết tính năng này đã deprecated và service mới từ 2026-08-28 không thể bật; cấu hình hiệu lực ở bài này được đặt và xác minh trong dashboard.
- Sau khi người dùng cho phép public networking, Railway tạo URL `https://k4-l3a-day12-nguyenminhhien-2a202602759-cloudser-production.up.railway.app` trỏ cổng 8080. HTTPS là đường gọi từ Internet; địa chỉ `railway.internal` chỉ dùng giữa các service trong Railway.
- Kết quả kiểm tra thật ngày 2026-09-28: `/health` → 200 `status=ok`; `/ready` → 200 `redis=true`; `/ask` thiếu khóa → 401; `/ask` có khóa từ cấu hình local → 200 có câu trả lời; 15 request cùng user trong 60 giây → 10 lần 200, 5 lần 429. Test `tests/test_cp5.py -v` đạt 9 pass; 4 test local fallback được skip vì đã dùng cloud. Kết quả và URL nằm trong `DEPLOYMENT.md`.
- Chạy tích hợp tất cả `tests/test_cp1.py` đến `tests/test_cp5.py` với Docker CLI trong PATH: 79 pass, 4 skip chỉ thuộc local fallback vì đã dùng cloud, 1 `StarletteDeprecationWarning`. Docker build thật và test cloud đều được chạy trong lần này.
- Railway hiện có 0 active sandbox và 1 sandbox đã Destroyed. Hai tài nguyên Online cho bài là Redis và web service. Người dùng giới hạn thao tác trong trial hiện có (30 ngày hoặc $5), không nâng cấp hoặc thêm phương thức thanh toán.
- Hai ảnh minh chứng đã có trong `screenshots/`: dashboard thấy Redis/web service Online và deployment successful; ảnh health thấy đúng domain public cùng JSON `status=ok`. Đã kiểm tra không thấy giá trị secret trong ảnh.

### 7. Giới hạn và cách hiểu kết quả
- Các test CP1/CP3/CP4 dùng fakeredis hoặc stub cho phần lớn kiểm tra; chúng xác nhận logic theo rubric. Smoke test Compose bổ sung bằng Redis thật, nhưng không chứng minh cloud đã chạy.
- Rate limit và cost guard hiện bám đúng thuật toán lab, chưa có giao dịch nguyên tử cho nhiều request đồng thời; `X-User-Id` là header do client cung cấp theo thiết kế bài, chưa phải định danh người dùng độc lập được xác minh.
- Cảnh báo `StarletteDeprecationWarning` xuất phát từ phiên bản thư viện test mới; các test checkpoint vẫn đạt. Chưa thay dependency chỉ để loại cảnh báo này.

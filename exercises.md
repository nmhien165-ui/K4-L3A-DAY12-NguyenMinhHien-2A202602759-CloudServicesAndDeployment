# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
> Các câu trả lời dưới đây là bản nháp dựa trên kết quả chạy và sự cố đã ghi
> nhận trong project; hãy sửa chi tiết nào không khớp trải nghiệm của bạn.
>
> Cách trả lời: viết câu trả lời ngay dưới từng câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Minh Hiển  Mã học viên: 2A202602759

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu quên đặt `AGENT_API_KEY` trên Railway, cấu hình không có giá trị mặc định
> nên app dừng ngay khi khởi động và Railway báo deploy/health check không đạt.
> Mình có thể sửa biến trước khi service nhận request. Nếu mặc định là
> `changeme`, app lại khởi động bình thường; ai biết giá trị mặc định có thể gọi
> `/ask`, làm lộ dữ liệu hoặc dùng hết ngân sách trước khi mình phát hiện.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> ```json
> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T10:39:52.325546+00:00", "user_id": "sv-test", "tokens_in": 4, "tokens_out": 36, "cost_usd": 2.22e-05}
> ```
>
> Mình lấy dòng này khi chạy test request `/ask` có ghi chi phí. Từ các trường
> JSON mình có thể (1) lọc và cộng `cost_usd` theo `user_id` để tìm người dùng
> tiêu nhiều nhất, và (2) đếm sự kiện theo `level`/`timestamp` để theo dõi lỗi
> theo thời gian. `print("đã trả lời xong")` không có các trường máy đọc được
> này.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (Dockerfile gốc `python:3.11`, build lại thành `agent:single`) | 1.73 GB (đo bằng `docker image ls`) |
| Multi-stage (`day12-agent:prod`) | 271 MB (đo bằng `docker image ls`) |

> Bản multi-stage nhỏ hơn khoảng 1.46 GB (khoảng 84%). Bản gốc dùng image
> Python đầy đủ và cài tất cả dependency trong cùng stage; multi-stage dùng
> `python:3.11-slim`, chỉ chuyển thư viện đã cài và source cần chạy sang runtime.
> Vì vậy runtime không mang theo các file/build tools chỉ cần lúc cài dependency.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile hiện copy `requirements.txt` và cài package trong stage `builder`
> trước khi copy source, nên sửa `app/main.py` không làm thay đổi các layer cài
> dependency; builder được dùng lại từ cache. Ở runtime, layer copy `/install`
> cũng được dùng lại, còn layer `COPY app` bị tạo lại và các lệnh phía sau nó
> phải chạy lại theo chuỗi cache. Nếu `COPY . .` đứng trước `pip install`, sửa
> một file source làm layer copy đổi, khiến pip install phải chạy lại dù
> `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu app có lỗ hổng cho phép chạy lệnh tùy ý, kẻ tấn công có thể chiếm quyền
> tiến trình trong container. Nếu container chạy root, tiến trình đó có quyền
> cao trong container và có thể khai thác lỗ hổng kernel hoặc cấu hình mount
> yếu để tìm cách thoát ra host. `USER appuser` chạy app với UID thường, nên
> lỗ hổng không tự động trao quyền root trong container; nó giảm quyền ban đầu
> và giới hạn thiệt hại, dù không loại bỏ mọi khả năng container escape.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi **20 request trong 2 giây**: gửi 10 request ngay trước mốc phút
> mới, rồi 10 request ngay sau khi bộ đếm reset ở giây 00. Fixed window đếm
> riêng hai phút lịch nên cho qua cả hai nhóm. Sliding window xét 60 giây gần
> nhất tại từng request nên không cho phép burst kiểu đó vượt quá 10.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit đếm số request trong một khoảng thời gian; cost guard cộng tiền
> theo user trong tháng. Ví dụ mình còn dưới 10 request/phút nhưng chi phí tháng
> đã vượt ngân sách: rate limit cho qua, cost guard trả 402. Ngược lại, người
> dùng gửi nhiều câu hỏi ngắn, rẻ và còn ngân sách nhưng vượt 10 request/phút:
> rate limit trả 429, cost guard vẫn còn cho phép.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối → probe gộp `/health` kiểm tra Redis và báo lỗi ở cả ba
> container → orchestrator hiểu nhầm process của cả ba container đã hỏng và
> khởi động lại chúng → container mới vẫn không kết nối được Redis nên lại
> thất bại/restart, làm gián đoạn toàn cụm dù Redis chỉ lỗi 30 giây. Tách probe
> giúp `/health` vẫn báo process còn sống; `/ready` báo chưa sẵn sàng để load
> balancer tạm ngừng gửi traffic, không restart cả cụm.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi lưu history trong Redis, mọi container đọc cùng danh sách nên các request
> kế tiếp nhìn thấy cùng lịch sử; `history_length` tăng 2 mỗi lượt hỏi vì mỗi
> lượt thêm một message user và một message assistant (ví dụ 0, 2, 4...). Nếu
> dùng dict trong RAM, mỗi container giữ một bản riêng. Request chuyển sang
> container chưa từng nhận user đó có thể lại thấy 0 hoặc số nhỏ hơn, nên kết
> quả phụ thuộc request được cân bằng vào container nào.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi kiểm tra CP5, Railway không áp dụng cấu hình mình trông đợi từ
> `railway.toml`; dashboard báo Config as Code cũ đã deprecated/không dùng được
> cho service mới. Mình kiểm tra phần cấu hình của service trên dashboard và
> thấy builder, biến môi trường, health check phải đặt tại đó. Mình cấu hình
> Dockerfile builder, `PORT`, `REDIS_URL` tham chiếu Redis và health check
> `/health` trong dashboard. Sau đó deployment báo successful, `/health` trả
> 200 và `/ready` trả 200 với Redis đã kết nối.

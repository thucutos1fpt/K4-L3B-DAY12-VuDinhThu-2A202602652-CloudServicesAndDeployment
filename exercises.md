# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay mỗi dòng gợi ý dưới từng câu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy Railway, nếu quên AGENT_API_KEY mà code dùng mặc định changeme thì service vẫn chạy và endpoint /ask có thể bị gọi trái phép. Việc không có mặc định làm Settings báo lỗi ngay khi khởi động, nên tôi phát hiện thiếu secret trước khi service phục vụ traffic.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Ví dụ log JSON: {"event":"ask_completed","level":"info","timestamp":"2026-09-29T04:00:00+00:00","user_id":"sv-test","cost_usd":0.0001}. Tôi có thể lọc theo user_id để biết ai dùng nhiều và cộng cost_usd để theo dõi chi phí; print thông thường không có cấu trúc ổn định để hệ thống truy vấn.

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
| 1 stage (bản đầu) | ... MB |
| Multi-stage | ... MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản single-stage mang cả lớp build và mọi dependency vào runtime, còn multi-stage chỉ copy dependency đã cài sang image chạy. Vì vậy phần chênh lệch chủ yếu là cache pip, công cụ build và file không cần thiết; image production của bài vẫn dưới giới hạn 500 MB.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa app/main.py, layer COPY requirements.txt và RUN pip install được lấy từ cache; layer COPY source và các layer sau chạy lại. Nếu COPY . . đứng trước pip install thì thay đổi nhỏ ở source cũng làm mất cache dependency và build chậm hơn nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Một lỗ hổng có thể cho kẻ tấn công thực thi lệnh trong container. Nếu process là root, quyền đó mở rộng phạm vi thiệt hại và tạo điều kiện leo thang trong môi trường container; USER appuser làm process chỉ có quyền tối thiểu trong filesystem và runtime.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với quota 10/phút, người dùng gửi 10 request lúc 10:00:59 và thêm 10 request lúc 10:01:01, tổng 20 request trong 2 giây. Sliding window nhìn 60 giây gần nhất nên chặn đợt thứ hai.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request theo thời gian, còn cost guard giới hạn tổng tiền theo user/tháng. Một request prompt rất dài có thể vẫn trong quota nhưng vượt budget; ngược lại nhiều request rẻ trong một phút có thể chạm rate limit dù tổng chi phí chưa vượt budget.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp health và ready rồi kiểm tra Redis, Redis mất 30 giây khiến cả ba container trả health fail. Orchestrator sẽ lần lượt restart chúng, load balancer mất toàn bộ instance dù process app vẫn khỏe. Tách /health giúp container còn sống, còn /ready chỉ rút instance khỏi traffic mới.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi scale 3 agent, history_length vẫn tăng tuần tự vì mọi instance đọc và ghi cùng Redis. Nếu dùng dict Python, mỗi container có RAM riêng nên request rơi vào container khác sẽ thấy history_length quay về 0 hoặc thấp hơn mong đợi.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi deploy tôi gặp là Railway chạy image cũ nên lifecycle.install vẫn ném NotImplementedError TODO. Tôi xem View logs để xác định file và dòng lỗi, kiểm tra code mới đã push lên main, sau đó bật Auto deploy và push commit mới để Railway build lại source hiện tại.

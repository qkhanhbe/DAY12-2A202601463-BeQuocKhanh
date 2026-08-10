# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên cloud, nếu quên đặt `AGENT_API_KEY` mà code dùng mặc định `changeme`, service vẫn chạy và endpoint `/ask` có thể bị người lạ gọi bằng khóa đó. Không có default làm Pydantic dừng service ngay lúc start, nên tôi phát hiện thiếu secret trước khi service nhận traffic hoặc phát sinh chi phí.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Log thực tế: `{"event":"ask_completed","level":"info","timestamp":"2026-08-10T04:40:47.055185+00:00","user_id":"rate-check","tokens_in":392,"tokens_out":43,"cost_usd":8.46e-05}`. Tôi có thể lọc/tổng hợp chi phí theo `user_id`, và tạo cảnh báo theo `cost_usd` hoặc tỷ lệ event lỗi. `print("đã trả lời xong")` không có field máy đọc được cho hai việc này.

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
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 296 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Chênh lệch chủ yếu là Python full image, toàn bộ build dependency và cache pip của bản một stage. Runtime multi-stage chỉ nhận package đã cài vào `/usr/local`, source `app`/`utils`, và base `python:3.11-slim`.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Sửa `app/main.py` chỉ làm Docker chạy lại `COPY app` và các layer sau nó; layer `COPY requirements.txt` cùng `pip install` dùng cache. Nếu `COPY . .` đứng trước `pip install`, thay đổi một ký tự source làm hỏng cache ở `COPY . .`, nên Docker phải cài lại toàn bộ dependency.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Lỗ hổng ứng dụng có thể cho kẻ tấn công chạy lệnh trong container. Nếu process là root, họ có quyền root trong container và có thể khai thác thêm lỗi cấu hình/kernel/mount để mở rộng ảnh hưởng sang host. `USER appuser` giảm quyền ngay sau khi process bị chiếm; lệnh chạy không còn quyền root để sửa file hệ thống hay cài tool đặc quyền.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa 20 request trong 2 giây: gửi 10 request lúc 10:00:59, rồi 10 request lúc 10:01:01. Bộ đếm theo phút reset tại 10:01:00 nên cả hai đợt đều dưới hạn mức. Sliding window giữ mọi request của 60 giây gần nhất nên đợt hai bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn nhịp gọi; cost guard giới hạn tổng tiền theo user/tháng. User gửi 10 request/phút đúng hạn mức nhưng mỗi request 50.000 token: rate limit cho qua, cost guard phải chặn khi vượt budget. Ngược lại, user spam 11 request rẻ trong vài giây khi mới tiêu 0 USD: cost guard còn cho qua nhưng rate limit trả 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối làm endpoint gộp trả 503 từ cả 3 container. Orchestrator coi 503 là lỗi liveness, restart cả 3 container; trong lúc Redis chỉ mất 30 giây thì toàn bộ instance bị rút/restart, có thể kéo dài outage. Tách `/health` giữ process sống, còn `/ready` chỉ bảo load balancer ngừng gửi traffic vào instance chưa dùng được.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Tôi chạy `docker compose up --scale agent=3` qua Nginx. `history_length` vẫn tăng thêm 2 sau mỗi lượt cho cùng `X-User-Id`, kể cả khi Nginx đổi instance; đây là vì Redis dùng chung. Nếu dùng dict Python, mỗi instance chỉ thấy lịch sử trong RAM của nó: khi request rơi vào instance khác, `history_length` có thể quay về 0 hoặc nhỏ hơn ngẫu nhiên.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi build thử bản Dockerfile một stage với file Dockerfile nằm ngoài build context, Docker báo `failed to xattr ... permission denied`. Tôi kiểm tra log build và thấy Docker đang đọc file ở `/tmp`; chuyển Dockerfile tạm vào đúng thư mục build context rồi build lại. Bản multi-stage chính thức build thành công, image 296 MB, `/health` và `/ready` đều 200 trên Docker Compose.

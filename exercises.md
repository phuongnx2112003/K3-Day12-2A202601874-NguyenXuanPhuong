# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Xuân Phượng  Mã học viên: 2A202601874

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên cloud, nếu quên set `AGENT_API_KEY` mà app có key mặc định như `changeme`, service vẫn chạy và người khác có thể đoán hoặc dùng key đó để gọi `/ask`. Việc bắt buộc biến này làm app lỗi ngay lúc khởi động, nên tôi phát hiện cấu hình sai trước khi service public nhận request và phát sinh chi phí.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một log tôi nhận được có dạng: `{"event":"ask_completed","level":"info","timestamp":"2026-08-10T03:03:34+00:00","user_id":"sv-test","tokens_in":3,"tokens_out":34,"cost_usd":0.00002055}`. Tôi có thể lọc/tổng hợp chi phí theo `user_id` để biết user nào tốn nhiều nhất, và đếm event theo `level` hoặc thời gian để cảnh báo tỷ lệ lỗi. Một dòng `print("đã trả lời xong")` không có các trường dữ liệu này để máy truy vấn.

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
| 1 stage (bản đầu) | khoảng 1.1 GB |
| Multi-stage | 183 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản một stage giữ Python image đầy đủ, cache pip và các thành phần chỉ phục vụ quá trình build. Bản multi-stage chỉ copy dependency đã cài sang runtime slim nên loại được phần môi trường build không cần khi chạy ứng dụng.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, layer `COPY requirements.txt` và `RUN pip install` vẫn được cache; Docker chỉ chạy lại các layer copy source và các layer sau đó. Nếu `COPY . .` đứng trước `pip install`, thay đổi một ký tự source sẽ làm layer copy đổi hash và Docker phải cài lại toàn bộ dependency.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Một lỗ hổng có thể cho phép kẻ tấn công chạy lệnh trong container. Nếu process là root, họ có quyền root trong container và có thêm cơ hội khai thác cấu hình/mount sai để mở rộng quyền ra host. `USER appuser` khiến process ứng dụng chỉ có quyền user thường, nên lệnh chạy được sau khi xâm nhập không có quyền quản trị mặc định.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi 20 request trong khoảng 2 giây: 10 request ở 10:00:59 và thêm 10 request ở 10:01:01. Bộ đếm theo phút reset khi sang phút mới nên đều cho qua, còn sliding window 60 giây vẫn thấy 10 request đầu và chặn nhóm sau.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong một khoảng thời gian; cost guard giới hạn tổng tiền theo tháng. Một user gửi không quá 10 request/phút nhưng mỗi request rất dài có thể qua rate limit và bị cost guard chặn. Ngược lại, user gửi dồn hơn 10 request/phút với request rẻ có thể bị rate limit chặn dù ngân sách tháng vẫn còn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối làm probe gộp trả 503 cho cả ba container. Orchestrator coi chúng unhealthy và restart đồng thời; load balancer không còn instance nào nhận traffic. Khi Redis hồi phục, các container phải khởi động lại nên một lỗi Redis ngắn đã thành gián đoạn toàn hệ thống. Vì vậy `/health` chỉ kiểm tra process, còn `/ready` mới kiểm tra Redis.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis, mỗi lần `/ask` lưu hai message nên request thứ hai nhìn thấy `history_length` là 2 dù có thể đi vào container khác. Nếu dùng dict Python, mỗi container có state riêng: history length sẽ có lúc là 0, lúc là 2 hoặc tăng không đều tùy load balancer gửi request vào instance nào.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi deploy Render, `/health` trả 200 nhưng `/ready` trả 503 với `{"status":"not ready","redis":false}`. Tôi đối chiếu log và cấu hình, nhận ra app cloud không thể dùng Redis local. Tôi tạo/kết nối Render Redis, gán connection string nội bộ vào `REDIS_URL`, rồi redeploy. Sau đó `/ready` trả 200 với `redis: true`.

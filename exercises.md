# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay phần trả lời mẫu dưới mỗi câu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Đình Liêm  Mã học viên: 2A202601421
---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Ví dụ khi deploy service lên cloud, tôi quên cấu hình biến môi trường AGENT_API_KEY. Nếu hệ thống để mặc định "changeme", service vẫn khởi động bình thường nên nhìn bên ngoài có vẻ deployment thành công. Chỉ đến khi người dùng gọi /ask thì request mới lỗi do API key không hợp lệ, khiến lỗi khó phát hiện và service có thể chạy sai trong một thời gian. Với cơ chế fail fast, service sẽ không khởi động ngay khi thiếu AGENT_API_KEY. Nhờ đó tôi biết deployment/configuration đang sai ngay từ đầu và sửa trước khi service nhận request thực tế.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

{"event":"ask_completed","level":"info","timestamp":"2026-08-10T04:36:33.981261+00:00","user_id":"cp5-test","tokens_in":376,"tokens_out":45,"cost_usd":8.34e-05}

Dòng log cho biết sự kiện ask_completed đã xảy ra, user nào thực hiện,
thời điểm, số token đầu vào/đầu ra và chi phí của request.

Hai việc có thể làm với dòng log này mà print("đã trả lời xong") không làm
được là:

1. Lọc và thống kê chi phí, số token hoặc số request theo user và thời gian.
2. Tạo cảnh báo tự động khi chi phí tăng cao, request lỗi hoặc lưu lượng
   request bất thường.

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
| 1 stage (bản đầu) | 1.73GB |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Multi-stage nhỏ hơn vì image cuối chỉ chứa runtime, dependency và source
cần thiết. Các thành phần build, cache pip và môi trường trung gian không
được copy sang image cuối. Bản single-stage giữ toàn bộ source, môi trường
cài đặt và thành phần không cần thiết nên thường lớn hơn.


---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Sau khi sửa một ký tự trong app/main.py và build lại, các layer trước bước
COPY source như FROM, WORKDIR, COPY requirements.txt và pip install được
dùng lại từ cache. Layer COPY source và các layer phía sau phải chạy lại.

Nếu đặt COPY . . trước RUN pip install thì mỗi lần sửa bất kỳ file source nào,
layer COPY bị thay đổi và Docker phải chạy lại pip install. Vì vậy build chậm
hơn và mất lợi ích cache dependency.
---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Nếu code Python có lỗ hổng cho phép kẻ tấn công thực thi lệnh, tiến trình
container chạy bằng root sẽ cho kẻ đó quyền root bên trong container. Nếu
container có thêm lỗ hổng escape hoặc quyền truy cập Docker socket, kẻ tấn công
có thể tiếp tục chiếm quyền cao trên máy host.

Lệnh USER appuser làm gián đoạn chuỗi này: tiến trình ứng dụng không còn quyền
root, nên việc khai thác lỗi chỉ nhận được quyền hạn thấp hơn.
---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Với rate limit 10 request/phút nhưng đếm theo phút đồng hồ, người dùng có thể
gửi 10 request lúc 10:00:59 và thêm 10 request lúc 10:01:00 hoặc 10:01:01.
Như vậy có thể gửi tối đa 20 request trong khoảng 2 giây liên tiếp.

Đây là lỗi cửa sổ trượt giả vì bộ đếm reset theo mốc phút thay vì đếm đúng
60 giây gần nhất.
---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit giới hạn số lượng request trong một khoảng thời gian. Cost guard
giới hạn tổng số tiền đã tiêu trong tháng.

Ví dụ rate limit cho qua nhưng cost guard chặn: user mới chỉ gửi 1 request,
chưa vượt giới hạn request/phút, nhưng request đó có chi phí 20 USD trong khi
ngân sách tháng chỉ còn 5 USD.

Ví dụ ngược lại: user gửi quá nhiều request nhỏ trong một phút nên bị rate
limit chặn, dù tổng chi phí đã tiêu vẫn còn rất thấp và chưa vượt ngân sách.
---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Ban đầu cả 3 container đều đang healthy và nhận traffic. Khi Redis mất kết nối,
nếu /health cũng kiểm tra Redis thì cả 3 container sẽ trả unhealthy.

Load balancer sẽ ngừng gửi traffic đến cả 3 container. Orchestrator sau đó
restart cả 3 container cùng lúc. Trong 30 giây Redis mất kết nối, toàn bộ cụm
không còn instance phục vụ. Khi Redis hoạt động lại, các container mới khởi
động lại và trở thành ready.
---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Khi gọi nhiều lần với cùng X-User-Id là sv-test, history_length tăng từ
12 lên 14, 16 rồi 18. Mỗi request tăng 2 vì hệ thống lưu cả câu hỏi của user
và câu trả lời của assistant. Điều này cho thấy lịch sử được lưu ở Redis,
nên các request hoặc container khác nhau vẫn đọc được cùng dữ liệu.

Nếu lưu lịch sử trong dict Python, khi request chuyển sang instance khác,
history_length có thể quay về 0 hoặc thay đổi không ổn định vì mỗi container
có bộ nhớ RAM riêng.
---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Khi chạy GitHub Actions, job “Deploy to Render” bị fail với exit code 3,
trong khi job Test và Build Docker image vẫn chạy được. Nguyên nhân là
secret `RENDER_DEPLOY_HOOK_URL` chưa được cấu hình hoặc chứa URL không hợp lệ,
nên lệnh curl không thể gọi Render Deploy Hook.

Tìm nguyên nhân bằng cách mở log của job Deploy to Render và thấy lỗi
xảy ra ngay ở bước gọi curl. Sau đó mình lấy Deploy Hook URL trong Render,
tạo secret cùng tên `RENDER_DEPLOY_HOOK_URL` trong GitHub Actions rồi chạy
workflow lại. Kết quả job deploy chạy thành công.

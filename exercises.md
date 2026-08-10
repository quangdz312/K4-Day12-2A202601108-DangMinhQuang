# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Dang Minh Quang Mã học viên: 2A202601108

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu quên đặt `API_TOKEN` trên production, fail-fast làm tiến trình dừng ngay và báo lỗi cấu hình; với mặc định `"changeme"`, service vẫn mở và kẻ khác có thể dùng token dễ đoán để gọi API, gây lộ dữ liệu hoặc phát sinh chi phí.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Ví dụ hợp lệ: `{"event":"chat_completed","severity":"INFO","ts":"2026-08-10T03:00:00+00:00","client_id":"sv01","prompt_tokens":12,"completion_tokens":8,"usd_cost":0.00002}`. Máy có thể (1) lọc/đếm `chat_completed` theo `client_id`, và (2) cộng `usd_cost` để lập dashboard hoặc cảnh báo; câu `print` không cung cấp các trường có cấu trúc đó.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản               | Dung lượng              |
| ----------------- | ----------------------- |
| 1 stage (bản đầu) | [đo bằng docker images] |
| Multi-stage       | [đo bằng docker images] |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Docker không khả dụng trong môi trường này nên cả hai số phải để `[đo bằng docker images]`. Về cấu trúc, bản multi-stage chỉ chép thư viện đã cài từ `builder` sang runtime `python:3.11-slim`, không mang theo cache/công cụ build và các lớp trung gian; vì chưa đo nên không khẳng định mức chênh lệch.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, các layer base, `COPY requirements.txt`, cài dependency và tạo user được dùng lại; từ `COPY . .` trở đi phải chạy lại. Nếu đặt `COPY . .` trước `RUN pip install`, mọi thay đổi source làm mất cache của layer cài thư viện, khiến build chậm dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi rủi ro là: lỗ hổng Python cho phép thực thi lệnh trong container → tiến trình root sửa filesystem/cấu hình hoặc khai thác lỗi runtime/mount/socket → thoát container và có quyền cao trên host. `USER app` cắt chuỗi ngay sau bước chiếm tiến trình: mã của kẻ tấn công chỉ có quyền user thường, giảm khả năng sửa tài nguyên đặc quyền và leo thang/thoát container.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> `WWW-Authenticate: Bearer` cho client biết cơ chế xác thực cần dùng và đáp ứng ngữ nghĩa của 401. Cùng thông báo `invalid or missing bearer token` cho thiếu header, sai scheme và sai token giúp phản hồi nhất quán, không tiết lộ manh mối cho người đang dò thông tin xác thực.

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Có giới hạn sức chứa, sau 10 phút xô vẫn chỉ có 10 token nên gửi được 10 request rồi request kế tiếp nhận 429. Bỏ `min(capacity, ...)`, 10 token ban đầu cộng khoảng 100 token nạp trong 10 phút thành khoảng 110 request, vì token được tích lũy vượt sức chứa.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

> Hạn mức $30/tháng cho phép sự cố đốt tối đa $30 và chỉ tự hồi phục khi sang tháng mới. Hạn mức $1/ngày giới hạn thiệt hại tối đa khoảng $1 trong ngày đó và tự hồi phục khi khóa ngân sách chuyển sang ngày UTC mới; đổi lại sự cố còn tồn tại có thể tiếp tục tiêu tối đa $1 mỗi ngày.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Thứ tự: Redis mất kết nối → probe gộp của cả 3 container trả lỗi → orchestrator vừa ngừng chuyển traffic vừa coi cả 3 tiến trình là chết → lần lượt restart chúng → các container mới vẫn probe lỗi vì Redis chưa hồi phục → cả cụm tiếp tục vòng lặp restart, gây mất dịch vụ dù code ứng dụng còn sống. Tách probe thì `/readyz` loại instance khỏi traffic, còn `/healthz` vẫn 200 nên không restart hàng loạt; Redis hồi phục thì readiness tự trở lại.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Quan sát thực tế tại môi trường làm bài: PowerShell báo `docker: The term 'docker' is not recognized as the name of a cmdlet, function, script file, or operable program`. Tôi chẩn đoán bằng `Get-Command docker`, kết quả không tìm thấy executable; cách khắc phục là cài/khởi động Docker Desktop và thêm Docker CLI vào `PATH`, rồi mở terminal mới và kiểm tra `docker version` trước khi build/deploy. Đây là bước chuẩn bị môi trường, không phải một lần deploy cloud đã được thực hiện; tôi không bịa kết quả triển khai.

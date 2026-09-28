# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trần Thị Như Ý  Mã học viên: 2A202602372

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Kịch bản fail fast này hữu ích: Khi thiếu agent api key, pydantic validation error và bị crash ngay từ đầu thì sẽ giúp mình phát hiện vấn đề ngay ở giai đoạn đầu, dễ fix, dễ bảo trì về sau. Ví dụ: cấu hình CI/CD pipeline, lỗi/thiếu hoặc quên set agent api key trong github secrets, thì fail fast sẽ báo error và mình vào fix ngay trước khi tới tay stakeholder. Ngược lại nếu để default "changeme" thì app vẫn chạy bình thường, đến khi bên stakeholder check thì bung bug, gây reputation damage. 

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T18:17:58.827054+00:00", "user_id": "sv-test", "tokens_in": 3, "tokens_out": 37, "cost_usd": 2.265e-05}
> INFO:     127.0.0.1:59697 - "POST /ask HTTP/1.1" 200 OK

> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T18:18:13.413304+00:00", "user_id": "sv-test", "tokens_in": 43, "tokens_out": 47, "cost_usd": 3.465e-05}
> INFO:     127.0.0.1:59703 - "POST /ask HTTP/1.1" 200 OK

> Với JSON log thì xem được tokens in/tokens out từ đó phân tích được mức dùng LLM, xem được cost để tính chi phí từng user theo ngày/tháng; so sánh chi phí giữa các user và có thể cảnh báo nếu có user vượt budget.
> Còn với print("đã trả lời xong") thì không làm được các chức năng như của JSON log đã liệt kê ở trên.
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
| 1 stage (bản đầu) | 288 MB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Chênh lệch: 17 MB. Phần chênh lệch này chủ yếu:

> APT package cache: khi pip install chạy trong single-stage, nó giữ lại các file tạm, header, và cache của thư viện hệ thống cần thiết khi build. Ở multi-stage, những thứ đó nằm ở builder stage, không được copy sang runtime stage, nên không tồn trong image cuối. 

> __pycache__/ .pyc: python tự biên dịch .pyc khi import. Ở single-stage các file này vẫn còn trong image. Ở multi-stage, stage mới bắt đầu sạch, chỉ copy source .py, không có .pyc

> Toolchain/pip internals: một số metadata và file temp của pip khi cài package bị loại ở multi-stage

> Lệch 17MB khi file image gốc là python:3.11-slim, nếu dùng base image đầy đủ (python:3.11) thì có thể lệch nhiều hơn. 
---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

```
PS F:\K4-L3A-TranThiNhuY-L3A202602372-CloudServicesAndDeployment> docker build -t day12-agent:prod .
[+] Building 2.1s (14/14) FINISHED                                                                 docker:desktop-linux 
 => [internal] load build definition from Dockerfile                                                               0.0s 
 => => transferring dockerfile: 2.43kB                                                                             0.0s 
 => [internal] load metadata for docker.io/library/python:3.11-slim                                                1.7s 
 => [auth] library/python:pull token for registry-1.docker.io                                                      0.0s 
 => [internal] load .dockerignore                                                                                  0.0s 
 => => transferring context: 336B                                                                                  0.0s
 => [internal] load build context                                                                                  0.0s
 => => transferring context: 1.20kB                                                                                0.0s
 => [builder 1/4] FROM docker.io/library/python:3.11-slim@sha256:e41613d42d4891e4930f79523f93f81bbc7632584ec65e36  0.0s
 => => resolve docker.io/library/python:3.11-slim@sha256:e41613d42d4891e4930f79523f93f81bbc7632584ec65e36ab055f41  0.0s
 => CACHED [builder 2/4] WORKDIR /app                                                                              0.0s
 => CACHED [builder 3/4] COPY requirements.txt .                                                                   0.0s 
 => CACHED [builder 4/4] RUN pip install --no-cache-dir --prefix=/install -r requirements.txt                      0.0s 
 => CACHED [runime 3/6] COPY --from=builder /install /usr/local                                                    0.0s 
 => CACHED [runime 6/6] RUN useradd --create-home --uid 10001 appuser                                              0.0s 
 => exporting to image                                                                                             0.2s 
 => => exporting layers                                                                                            0.0s 
 => => exporting manifest sha256:d9fea831408c2b71b4b33b3814aeb690c5e7265e16016d9c543621789d4ab05f                  0.0s 
 => => exporting config sha256:15d59e7ef8606c55d0d0b9d0d8b0249521cd50313c9c7c747848c93a67a648ad                    0.0s 
 => => exporting attestation manifest sha256:4c319cdfcdd03fd814b4127f1b7a8244531ca5a77ea0cf0d71eaad059bae3803      0.0s 
 => => exporting manifest list sha256:15c9c3404efe9f64afc848bb4be26c848531180765de3d1d0fe61dec1de259a6             0.0s 
 => => naming to docker.io/library/day12-agent:prod                                                                0.0s 
 => => unpacking to docker.io/library/day12-agent:prod                                                             0.0s 

View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/5t76i3zauxlc68k6dtm31wkhd
PS F:\K4-L3A-TranThiNhuY-L3A202602372-CloudServicesAndDeployment> docker build -t day12-agent:prod .
[+] Building 2.5s (13/13) FINISHED                                                                 docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 2.43kB                                                                             0.0s
 => [internal] load metadata for docker.io/library/python:3.11-slim                                                1.2s 
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 336B                                                                                  0.0s 
 => [internal] load build context                                                                                  0.0s 
 => => transferring context: 6.11kB                                                                                0.0s 
 => [builder 1/4] FROM docker.io/library/python:3.11-slim@sha256:e41613d42d4891e4930f79523f93f81bbc7632584ec65e36  0.0s 
 => => resolve docker.io/library/python:3.11-slim@sha256:e41613d42d4891e4930f79523f93f81bbc7632584ec65e36ab055f41  0.0s 
 => CACHED [builder 2/4] WORKDIR /app                                                                              0.0s 
 => CACHED [builder 3/4] COPY requirements.txt .                                                                   0.0s 
 => CACHED [builder 4/4] RUN pip install --no-cache-dir --prefix=/install -r requirements.txt                      0.0s 
 => CACHED [runime 3/6] COPY --from=builder /install /usr/local                                                    0.0s 
 => [runime 4/6] COPY app/ ./app/                                                                                  0.0s 
 => [runime 5/6] COPY utils/ ./utils/                                                                              0.1s 
 => [runime 6/6] RUN useradd --create-home --uid 10001 appuser                                                     0.5s 
 => exporting to image                                                                                             0.5s 
 => => exporting layers                                                                                            0.2s 
 => => exporting manifest sha256:5aee85167dc29ed1da8bbbd4aed36dfab7173b3f97a0d723cd8fc78598715439                  0.0s 
 => => exporting config sha256:a1dc3f3de5e3144cf05cd6f6d5e92414f6942b8cb999748e62bd0d763afbfb6f                    0.0s 
 => => exporting attestation manifest sha256:b96b8134b833fdcb545c990110102d89d5171f92a7cc6be18efb881fb16ac9a4      0.0s 
 => => exporting manifest list sha256:6d089a66e9eff80053247ad879a714ae9d7b0da77e906b7327b6eae96b4572c4             0.0s 
 => => naming to docker.io/library/day12-agent:prod                                                                0.0s 
 => => unpacking to docker.io/library/day12-agent:prod                                                             0.1s 

View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/t0g328l2iirsrspo0g4xxs4m8
PS F:\K4-L3A-TranThiNhuY-L3A202602372-CloudServicesAndDeployment>
```

> Lần 1: docker build không đổi file mất 2.1s. Tất cả CACHED kể cả COPY app/, COPY utils/, RUN useradd. Không có gì phải chạy lại.

> Lần 2: sau khi thêm comment #cache test vào app/main.py, docker build mất 2.5s: builder 2/4 workdir cached không đổi,  builder 3/4 COPY requirements.txt cache không đổi, builder 4/4 RUN pip install cached nên không phải cài lại library, runime 3/6 COPY --from=builder/install  lấy từ builder nên cũng cache,  runime 4/6 COPY app/ do vừa đổi checksum trong app/main.py nên bị vỡ cache và mọi layer sau layer này đều phải chạy lại.

> Nếu đặt COPY . . lên trước RUN pip install (ví dụ COPY . . rồi mới RUN pip install -r requirements.txt), việc sửa 1 dòng code cũng làm COPY vỡ cache → RUN pip install ngay sau nó vỡ theo → phải tải và cài lại toàn bộ dependency (~15-24s như khi build agent:single). Đó là lý do Dockerfile phải COPY requirements.txt và RUN pip install TRƯỚC khi COPY app/.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Khi container chạy root, một bug RCE trong code Python cho phép attacker thực thi lệnh bên trong container với tư cách root (uid=0). Với quyền root, attacker có thể khai thác kernel hoặc mount đĩa host từ /dev/sda để thoát container và lên thẳng host, đọc được .env của mọi container và chiếm quyền máy chủ. Lệnh USER appuser cắt đứt chuỗi ở bước breakout: từ lúc RCE xảy ra, process chỉ chạy dưới uid=10001 (không phải root), nên mọi syscall đòi đặc quyền root đều bị kernel từ chối (EPERM). Attacker vẫn khai thác được lỗ hổng trong code nhưng bị nhốt trong container với quyền hạn tối thiểu — thiệt hại không vượt ra ngoài.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Nếu đếm theo phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa 12 request trong 2 giây liên tiếp: gửi 1 request ở giây 59 của phút cũ (vẫn nằm trong phút cũ, bộ đếm = 10/10), sau đó phút mới bắt đầu → bộ đếm reset về 0 → gửi ngay 10 request ở giây 00-09 phút mới, rồi gửi thêm 1 request ở giây 01 → tổng cộng 1+10+1 = 12 request. Sliding window tránh được điều này vì cửa sổ trượt liên tục, không có khoảng reset bất ngờ.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

```
Tình huống rate limit cho qua, cost guard chặn:
  - User gửi 1 request rất lớn, chứa prompt dài + history đầy đủ → rate limit: đây là request đầu tiên, cho
    qua
  - Cost guard: LLM trả lời xong, cost_usd = $0.05, tổng tháng đã là $10.05 → vượt budget → bị chặn 402

Tình huống ngược lại — cost guard cho qua, rate limit chặn:
  - User gửi 10 request liên tục trong 30 giây → rate limit: đã đạt 10/phút → bị chặn 429
  - Cost guard: mỗi request chỉ tốn $0.00003, tổng tháng mới $0.3 → dưới budget → cho qua
```

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

>  Khi gộp /health và /ready, cả hai đều kiểm tra Redis. Khi Redis mất kết nối: (1) Container A gọi Redis timeout → trả 503; (2) Container B, C cũng trả 503; (3) Load balancer thấy tất cả 503 → ngừng gửi traffic đến cả cụm 3 container; (4) Website hoàn toàn down dù cả 3 app vẫn sống; (5) Sau 30 giây Redis reconnect, nhưng load balancer cần thêm 15-30 giây để detect lại → tổng downtime 45-60 giây. Nếu tách /health (không kiểm tra Redis, luôn 200) và /ready (kiểm tra Redis, trả 503 khi Redis chết), LB vẫn gửi traffic vì /health tốt, app có thể trả lời request không cần Redis, downtime giảm đáng kể.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi dùng Redis, history_length tăng dần: 0, 1, 2, 3... vì cả 3 container đều đọc/ghi cùng một store Redis. Nếu dùng dict Python thay vì Redis, mỗi container có dict riêng biệt, không chia sẻ được với nhau. Container 1 lưu message vào dict của Container 1, nhưng khi request tiếp theo được load balancer chuyển sang Container 2, Container 2 không thấy message đó → history_length luôn nhảy lung tung giữa 0 và 1, không bao giờ tăng đúng. Đó là lý do stateful application cần external store thay vì in-memory dict.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi gặp: Railway deploy xong nhưng service liên tục restart, deploy logs hiện "Application failed to respond within healthcheck timeout". Tôi tìm nguyên nhân bằng cách vào Dashboard → Service → Deploy Logs → thấy app bind vào port 8000 nhưng Railway gán PORT=8001 qua biến môi trường → app không nghe đúng cổng. Cách sửa: đổi CMD trong Dockerfile từ --port 8000 thành --port ${PORT:-8000} để app đọc biến $PORT từ platform, với fallback 8000 khi chạy local.

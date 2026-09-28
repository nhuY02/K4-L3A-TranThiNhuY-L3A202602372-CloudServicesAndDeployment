# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Trần Thị Như Ý |
| Mã học viên | 2A202602372 |
| Repo | https://github.com/nhuY02/K4-L3A-TranThiNhuY-L3A202602372-CloudServicesAndDeployment.git |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-production-8920.up.railway.app |
| Platform | Railway |
| Ngày deploy | 28.09.2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis service `day12-redis` trên Railway, dùng reference URL nội bộ |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-production-8920.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-production-8920.up.railway.app/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-production-8920.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-production-8920.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-production-8920.up.railway.app/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
PS C:\WINDOWS\system32> curl.exe -i https://day12-agent-production-8920.up.railway.app/health
HTTP/1.1 200 OK
Content-Type: application/json
Date: Mon, 28 Sep 2026 14:56:06 GMT
Server: railway-hikari
x-railway-request-id: 7LUP1o6OTPme63ZQn6XIxQ
Content-Length: 57
x-hikari-trace: hkg1.aebn
x-railway-edge: hkg1
Connection: keep-alive

{"status":"ok","service":"day12-agent","version":"1.0.0"}

---
PS C:\WINDOWS\system32> curl.exe -i https://day12-agent-production-8920.up.railway.app/ready
HTTP/1.1 200 OK
Content-Type: application/json
Date: Mon, 28 Sep 2026 15:12:03 GMT
Server: railway-hikari
x-railway-request-id: B8obywA_QzqYW_ja0_TJvA
Content-Length: 31
x-hikari-trace: hkg1.aebn
x-railway-edge: hkg1
Connection: keep-alive

{"status":"ready","redis":true}

---
PS C:\WINDOWS\system32> try {
  Invoke-WebRequest -Method Post     -Uri "$url/ask"
    -ContentType "application/json; charset=utf-8" `
    -Body $bodyBytes
} catch {
  $.Exception.Response.StatusCode
  $.ErrorDetails.Message
}
Unauthorized
{"detail":"invalid or missing API key"}
PS C:\WINDOWS\system32>

---
PS C:\WINDOWS\system32> $secureKey = Read-Host "Nhập API key mới" -AsSecureString
Nhập API key mới: *******************************************
PS C:\WINDOWS\system32> $apiKey = [System.Net.NetworkCredential]::new("", $secureKey).Password
PS C:\WINDOWS\system32> $headers = @{
  "X-API-Key" = $apiKey
  "X-User-Id" = "sv-test"
}
PS C:\WINDOWS\system32>
PS C:\WINDOWS\system32> Invoke-RestMethod -Method Post `
  -Uri "$url/ask"   -ContentType "application/json; charset=utf-8"
  -Headers $headers `
  -Body $bodyBytes



answer         : CÃ¢u há»i hay. test thÆ°á»ng ÄÆ°á»£c giáº£i quyáº¿t báº±ng cÃ¡ch chuáº©n hÃ³a mÃ´i trÆ°á»ng cháº¡y: cÃ¹ng
                 má»t image cháº¡y giá»ng nhau á» laptop vÃ trÃªn cloud.
user_id        : sv-test
history_length : 0
cost_usd       : 1.995E-05
tokens         : @{in=1; out=33}
PS C:\WINDOWS\system32>

---
PS C:\WINDOWS\system32> $secureKey = Read-Host "Nhập AGENT_API_KEY" -AsSecureString
Nhập AGENT_API_KEY: *******************************************
PS C:\WINDOWS\system32> $apiKey = [System.Net.NetworkCredential]::new("", $secureKey).Password
PS C:\WINDOWS\system32>
PS C:\WINDOWS\system32> $headers = @{
>>   "X-API-Key" = $apiKey
>>   "X-User-Id" = "sv-rate-test"
>> }
PS C:\WINDOWS\system32> $bodyBytes = [System.Text.Encoding]::UTF8.GetBytes('{"question":"test"}')
PS C:\WINDOWS\system32>
PS C:\WINDOWS\system32> for ($i = 1; $i -le 15; $i++) {
>>   try {
>>     $response = Invoke-WebRequest -Method Post `
>>       -Uri "$url/ask" `
>>       -ContentType "application/json; charset=utf-8" `
>>       -Headers $headers `
>>       -Body $bodyBytes
>>     [int]$response.StatusCode
>>   } catch {
>>     if ($_.Exception.Response) {
>>       [int]$_.Exception.Response.StatusCode
>>     } else {
>>       "ERROR"
>>     }
>>   }
>> }

Security Warning: Script Execution Risk
Invoke-WebRequest parses the content of the web page. Script code in the web page might be run when the page is parsed.
      RECOMMENDED ACTION:
      Use the -UseBasicParsing switch to avoid script code execution.

      Do you want to continue?

[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
200
200
200
200
200
200
200
200
200
200
429
429
429
429
429
PS C:\WINDOWS\system32>

```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

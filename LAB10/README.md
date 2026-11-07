
# BÁO CÁO THỰC HÀNH LAB 10
## Tấn công Remote DNS Cache Poisoning (Mô phỏng kỹ thuật Kaminsky) & Giải pháp Hardening trên BIND9 + pfSense

---

## 1. Thông tin sinh viên
- **Họ và tên:** Vũ Bá Lực
- **MSSV:** 1150080104
- **Lớp:** 11_ĐH_THMT
- **Học phần:** An toàn và Bảo mật Thông tin

---

## 2. Mục tiêu bài LAB
- **Hiểu bản chất Remote DNS Attack:** Phân biệt rõ sự khác nhau giữa Local DNS Spoofing (bắt lén gói tin trên cùng mạng nội bộ) và Remote DNS Cache Poisoning (tấn công mù từ xa qua Firewall).
- **Mô phỏng kỹ thuật Kaminsky (2008):** Thực hiện kỹ thuật vượt qua cơ chế Cache bằng cách liên tục truy vấn tên miền ngẫu nhiên không tồn tại, kết hợp gửi phản hồi giả mạo chứa bản ghi ủy quyền `NS` và `Glue A` để chiếm quyền quản lý toàn bộ Zone tên miền `example.lab10.test`.
- **Phân vùng mạng với pfSense:** Thiết lập phân vùng `ATTACKNET` (`10.10.0.0/24`) độc lập với `DNSLAB` (`10.9.0.0/24`) và `DMZ` (`172.16.6.0/24`), cấu hình Firewall Rule nghiêm ngặt cô lập Attacker.
- **Phòng thủ & Hardening:** Khôi phục dịch vụ BIND9 hiện đại, quan sát và chứng minh hiệu quả của cơ chế **Source-Port Randomization** (ngẫu nhiên hóa cổng nguồn) và **Recursion ACL** (giới hạn quyền truy vấn đệ quy).

---

## 3. Môi trường & Công cụ

| Thành phần | Địa chỉ IP | Vai trò / Cấu hình |
|------------|:----------:|-------------------|
| **VirtualBox** | Host | Phiên bản 7.0.x / 7.2.x |
| **LucVu-pfSense** | `10.9.0.1` / `10.10.0.1` / `172.16.6.1` | Firewall / Gateway kết nối 3 phân vùng mạng: DNSLAB, ATTACKNET, DMZ |
| **DNS-SERVER** | `10.9.0.10` | Ubuntu Server chạy BIND 9.18 (Baseline/Hardening) & `vuln_resolver_lab10.py` (Attack Phase) |
| **DMZ-Web / AUTH-DNS** | `172.16.6.10` & `172.16.6.20` | Web server gốc (`172.16.6.10:80`) và Authoritative DNS Relay (`172.16.6.20:53`) |
| **VICTIM** | `10.9.0.100` | Ubuntu Client đóng vai trò người dùng nạn nhân |
| **Kali-Lab9 (Attacker)** | `10.10.0.200` | Kali Linux nằm tại ATTACKNET chạy Evil DNS (`:53`), Fake Web (`:80`) và script Kaminsky |

---

## 4. Kết quả kiểm thử

| STT | Kịch bản kiểm thử | Mô tả chi tiết | Kết quả |
|:---:|:------------------|:--------------|:-------:|
| 1 | **Baseline VICTIM** | VICTIM truy vấn `www.example.lab10.test` qua BIND9 | **PASS** (Trả về IP gốc `172.16.6.10` và tải đúng web DMZ) |
| 2 | **Remote Proof (Sniff Test)** | Bật `tcpdump` trên Kali khi Victim truy vấn DNS | **PASS** (Kali hoàn toàn không bắt được gói tin nào giữa Resolver và AUTH-DNS) |
| 3 | **Kaminsky Attack Simulation** | Kali chạy `kaminsky_remote_lab10.py` đoán TXID 16-bit | **PASS** (Terminal Kali hiện `SUCCESS`, AUTH-DNS log `[HIT]`, Resolver log `[CACHE]`) |
| 4 | **Victim Redirection** | VICTIM phân giải lại tên miền sau khi cache bị poison | **PASS** (Bị chuyển hướng sang IP Attacker `10.10.0.200`) |
| 5 | **Fake Site Verification** | VICTIM truy cập web qua lệnh `curl` | **PASS** (Tải về mã nguồn trang Web giả mạo của Kali) |
| 6 | **Packet Capture Analysis** | Kiểm tra file bắt gói `.pcap` trên DNS-SERVER | **PASS** (Thấy rõ `10.9.0.10:33333 -> 172.16.6.20:53` và gói Forged Response ngược lại) |
| 7 | **Source-Port Randomization** | Khởi động lại BIND9 và tạo truy vấn từ Victim | **PASS** (Cổng nguồn gửi đi đổi ngẫu nhiên liên tục, không còn cố định 33333) |
| 8 | **Recursion ACL Restriction** | Kali thử gửi truy vấn đệ quy đến DNS-SERVER | **PASS** (BIND9 trả về `status: REFUSED`, chặn Open Resolver) |

---

## 5. Bảng đối chiếu Trước và Sau tấn công

| Nội dung kiểm thử | Trước khi tấn công (Baseline) | Sau khi đầu độc (Poisoned) | Sau khi Hardening (Restore) |
| :--- | :---: | :---: | :---: |
| **Phân giải `www.example.lab10.test`** | `172.16.6.10` (Web gốc) | `10.10.0.200` (Attacker) | `172.16.6.10` (Web gốc) |
| **DNS Upstream của Resolver** | `172.16.6.20` (AUTH-DNS hợp lệ) | `10.10.0.200` (Evil DNS của Kali) | `172.16.6.20` (AUTH-DNS hợp lệ) |
| **Nội dung Website hiển thị** | DMZ-WEB hợp lệ | LAB 10 Fake Site (Kali) | DMZ-WEB hợp lệ |
| **Cổng nguồn Upstream (Source Port)** | Ngẫu nhiên (Random Ephemeral) | Cố định `33333` (Lỗ hổng mô phỏng) | Ngẫu nhiên (Random Ephemeral) |
| **Quyền Recursion của Kali** | Bị từ chối (`REFUSED`) | Không áp dụng (Vuln Resolver) | Bị từ chối (`REFUSED`) |
| **Khả năng nghe lén (Sniff) của Kali** | Không (Bị pfSense cô lập) | Không (Dựa trên dự đoán TXID) | Không (Bị pfSense cô lập) |

---

## 6. Vấn đề gặp phải và cách khắc phục

> **Lỗi 1:** VICTIM bị lỗi SERVFAIL (`No Reachable Authority`) ngay từ bước kiểm tra Baseline.
>
> **Nguyên nhân:** Script `auth_dns_relay_lab10.py` trên DMZ-Web mặc định kiểm tra cứng điều kiện cổng nguồn `RESOLVER_PORT = 33333`. Tuy nhiên ở bước Baseline, DNS-SERVER đang chạy BIND9 với cơ chế ngẫu nhiên hóa cổng nguồn (ví dụ port 56646, 48470), khiến script trên DMZ âm thầm drop gói tin.
>
> **Khắc phục:** Tách điều kiện kiểm tra trong `auth_dns_relay_lab10.py`: Cho phép trả lời ngay lập tức tên miền baseline `www.example.lab10.test` bất kể cổng nguồn nào, chỉ áp dụng điều kiện cổng 33333 cho pha tấn công:
> ```python
> if qname == WWW:
>     time.sleep(0.15); sock.sendto(legit_response(q).to_wire(), addr)
>     print(f'[WWW] legitimate {WWW} -> {LEGIT_WEB}'); continue
> if addr[1] != RESOLVER_PORT: continue
> ```

> **Lỗi 2:** Lỗi `Lookup failed due to system error: No route to host` trên VICTIM.
>
> **Nguyên nhân:** Khi khởi động lại hệ thống, router `LucVu-pfSense` hoặc `DNS-SERVER` chưa được bật, khiến mạng nội bộ bị ngắt quãng, máy Victim không tìm thấy đường đi tới Gateway.
>
> **Khắc phục:** Đảm bảo thứ tự bật đủ cả 5 máy ảo trong phòng Lab:
> 1. `LucVu-pfSense` (Router)
> 2. `DMZ-Web` (AUTH-DNS)
> 3. `DNS-SERVER` (Resolver)
> 4. `Kali-Lab9` (Attacker)
> 5. `VICTIM` (Client)

> **Lỗi 3:** Khởi động lại máy ảo DMZ-Web thì service DNS bị mất IP phụ `172.16.6.20`.
>
> **Nguyên nhân:** Lệnh gán IP phụ bằng `ip addr add` chỉ lưu tạm thời trong bộ nhớ RAM. Khi tắt máy (Power Off), card mạng `enp0s3` bị reset về IP mặc định `172.16.6.10`.
>
> **Khắc phục:** Mỗi khi mở lại máy DMZ-Web, cần gán lại IP trước khi chạy script:
> ```bash
> sudo ip addr add 172.16.6.20/24 dev enp0s3
> cd /media/sf_04_AUTH_DNS_RELAY && sudo python3 auth_dns_relay_lab10.py
> ```

> **Lỗi 4:** Kali chạy `evil_dns_lab10.py` báo lỗi `OSError: [Errno 99] Cannot assign requested address`.
>
> **Nguyên nhân:** Card mạng của Kali chưa được cấu hình IP `10.10.0.200` của mạng `ATTACKNET` nên script không thể bind và mở cổng 53 trên IP này.
>
> **Khắc phục:** Chạy script thiết lập mạng có sẵn để cấu hình tự động:
> ```bash
> sudo bash /media/sf_03_ATTACKER/setup_kali_lab10.sh
> ```

> **Lỗi 5:** Lệnh `curl` trên VICTIM sau khi poison trả về `Error code: 404 - File not found`.
>
> **Nguyên nhân:** Tấn công DNS đã thành công (trỏ về `10.10.0.200`), tuy nhiên Web Server Python trên Kali được khởi động ở thư mục chưa có file `index.html`.
>
> **Khắc phục:** Tạo thư mục web chuẩn và chép file mẫu vào trước khi chạy máy chủ HTTP:
> ```bash
> mkdir -p ~/LAB10/fake_site
> cp /media/sf_03_ATTACKER/fake_index.html ~/LAB10/fake_site/index.html
> cd ~/LAB10/fake_site && sudo python3 -m http.server 80
> ```

---

## 7. Checklist nộp bài

- [x] Ảnh cấu hình pfSense: Interface `ATTACKNET 10.10.0.1/24` và bảng Firewall Rules cô lập.
- [x] Ảnh Kali nhận IP `10.10.0.200`, card mạng Internal `attack-net` với `Promiscuous Mode = Deny`.
- [x] Ảnh Baseline từ VICTIM: `www.example.lab10.test` phân giải ra IP gốc `172.16.6.10`.
- [x] Ảnh chứng minh Remote Attack: `tcpdump` trên Kali hoàn toàn im lặng khi Victim gửi query.
- [x] Ảnh log đồng bộ khi tấn công: DMZ hiện `[PENDING]` & `[HIT]`, Resolver hiện `[CACHE]`, Kali hiện `SUCCESS`.
- [x] Ảnh bắt gói mạng (`.pcap`): Thấy gói tin upstream `10.9.0.10:33333 -> 172.16.6.20:53` và gói Forged Response.
- [x] Ảnh hậu quả trên VICTIM: Tên miền trỏ về `10.10.0.200` và tải về giao diện Fake Site.
- [x] Ảnh Hardening: BIND9 hoạt động trở lại, source port ngẫu nhiên và từ chối recursion từ Kali (`REFUSED`).
- [x] Bảng so sánh Trước / Sau tấn công và trả lời đầy đủ 12 câu hỏi phân tích.

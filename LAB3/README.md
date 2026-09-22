# BÁO CÁO THỰC HÀNH LAB 3: THREATS, VULNERABILITIES AND ATTACKS

## 1. THÔNG TIN SINH VIÊN & BÀI THỰC HÀNH
* **Họ và tên:** Vũ Bá Lực
* **Mã số sinh viên (MSSV):** 1150080104
* **Lớp:** 11_ĐH_THMT
* **Học phần:** An toàn và Bảo mật Thông tin (Cybersecurity)
* **Tên bài thực hành:** LAB 3 – Threats, Vulnerabilities, and Defensive Analysis in Enterprise Endpoint Environments

---

## 2. PHIÊN BẢN MÔI TRƯỜNG & CÔNG CỤ (ENVIRONMENT BASELINE)
* **Hệ điều hành máy ảo (Guest OS):** Windows Server 2025 Datacenter Evaluation (Chạy trên VMware Workstation)
* **Chế độ mạng chính (Isolation):** VMware Host-only Network (Chỉ tạm thời chuyển sang NAT có kiểm soát trong TH5 để kiểm tra kết nối HTTPS ngoài Internet, sau đó đưa ngay về Host-only)
* **Ngôn ngữ thực thi & Môi trường runtime:**
  * Windows PowerShell 5.1 / PowerShell 7 (Run as Administrator)
  * Python 3.14.x
* **Bộ công cụ Sysinternals & Giám sát:**
  * Microsoft Sysmon v15.22 (Tích hợp schema cấu hình `sysmon-lab.xml`)
  * Sysinternals Autoruns v14.3
  * Sysinternals Process Explorer v17.06
* **Công cụ phân tích mạng:**
  * Wireshark v4.6.8
  * Npcap Driver v1.88 (Bật tính năng Loopback Traffic Capture)
* **Thư mục tài nguyên & Lưu trữ bằng chứng:**
  * Thư mục dự án/assets: `C:\LAB3_Threats_Assets\lab3_assets\`
  * Thư mục lưu bằng chứng kiểm thử: `C:\LAB3\Evidence\`

---

## 3. CÁCH DỰNG VÀ CHUẨN BỊ MÔI TRƯỜNG (SETUP INSTRUCTIONS)
1. **Thiết lập thư mục làm việc:** Tạo cấu trúc thư mục chuẩn `C:\LAB3\Evidence`, `C:\LAB3\Tools`, và giải nén gói bài tập vào `C:\LAB3_Threats_Assets\lab3_assets\`.
2. **Cài đặt Npcap cho Wireshark:**
   * Cài đặt bộ cài `npcap-1.88.exe` từ thư mục Downloads.
   * **Bắt buộc kích hoạt:** Tích chọn `Support loopback traffic ("Npcap Loopback Adapter")` để hỗ trợ bắt gói tin trên loopback `127.0.0.1`.
3. **Cài đặt cấu hình giám sát Sysmon:**
   * Triển khai Sysmon 64-bit với tệp luật an toàn:  
     `Sysmon64.exe -accepteula -i C:\LAB3_Threats_Assets\lab3_assets\sysmon-lab.xml`
4. **Cấu hình cách ly mạng:** Thiết lập card mạng ảo về **Host-only** để đảm bảo an toàn tuyệt đối, tránh phát tán traffic ra mạng thực tế.

---

## 4. CÁC TÌNH HUỐNG ĐÃ THỰC HIỆN & KẾT QUẢ KIỂM THỬ (TEST RESULTS)

| STT | Tình huống (Scenario) | Mục tiêu & Cơ chế kiểm chứng | Bằng chứng (Evidence) | Kết quả |
| :---: | :--- | :--- | :--- | :---: |
| **TH1** | **Môi trường & Baseline** | Ghi nhận cấu hình hệ điều hành, tài nguyên phần cứng, phiên bản phần mềm bảo vệ và trạng thái ban đầu của hệ thống. | `H1_SystemInfo.png`, `H2_Defender_Status.png` | **PASS** |
| **TH2** | **Malware & AV Evasion (EICAR)** | Ghi nhận chuỗi kiểm thử EICAR, kích hoạt tính năng Real-time Protection và Scan của Windows Defender; phân tích Event ID 1116 trong Event Viewer và Protection History. | `H3_EICAR_Console.png`, `H4_ProtectionHistory_EICAR.png` | **PASS** |
| **TH3** | **Password Attacks & Keylogging Analysis** | Bật chính sách Audit Logon, tạo tài khoản thử nghiệm `lab3user`, mô phỏng các lần đăng nhập thành công (Event 4624/4648) và thất bại (Event 4625); thực hiện chu trình đổi mật khẩu (Credential Rotation) và phân tích nguy cơ Keylogger. | `H5_Event4625.png`, `auth_events_before_rotation.txt` | **PASS** |
| **TH4** | **Backdoor, Persistence & Listener** | Cài đặt Sysmon theo dõi tiến trình (Event 1); cắm Persistence lành tính qua Registry Run Key (`LAB3_Run_Demo`) và Scheduled Task (`LAB3_Persistence_Demo`); khởi chạy HTTP listener trên cổng `127.0.0.1:8080`, ánh xạ PID qua Process Explorer. | `H6_Sysmon_Event1.png`, `H7_Autoruns_LAB3_Run_Demo.png`, `H8_ProcessExplorer_Python.png` | **PASS** |
| **TH5** | **Sniffing & MITM: So sánh HTTP vs HTTPS** | Bắt gói tin Loopback phát hiện dữ liệu bản rõ (plaintext - chuỗi `TRAINING_ONLY`) qua HTTP 8080; chuyển NAT bắt gói cổng 443 chứng minh payload được mã hóa bảo vệ (Encrypted Application Data) qua TLS. | `H9_HTTP_Plaintext.png`, `H10_TLS_443.png` | **PASS** |
| **TH6** | **DoS / DDoS / Mail Bombing** | Chạy kiểm thử tải nội bộ (`local_load_test.py` 50 requests/5 workers) tới `127.0.0.1:8080`, quan sát tăng tải CPU và độ trễ dịch vụ; phân tích dữ liệu tấn công phân tán (Botnet) và cơ chế lạm dụng hàng loạt email. | `H11_DoS_LoadTest.png`, `H12_DDoS_Analysis.png` | **PASS** |
| **TH7** | **Social Engineering & Phishing Analysis** | Phân tích thông điệp email lừa đảo mẫu (Sender Spoofing, tính cấp bách Urgency, Hyperlink giả mạo cổng đăng nhập để đánh cắp định danh); phân tích các yếu tố cá nhân hóa trong Spear Phishing. | `H13_Phishing_Analysis.png` | **PASS** |
| **TH8** | **Cô lập, Cleanup, Phục hồi & Kiểm tra lại** | Buộc dừng HTTP listener cổng 8080, xóa bỏ khóa Registry Run `LAB3_Run_Demo`, hủy Scheduled Task, vô hiệu hóa tài khoản `lab3user`; kiểm chứng cổng 8080 đã đóng hoàn toàn và cập nhật mã băm toàn vẹn SHA-256. | `H14_Cleanup_Verification.png`, `evidence_sha256.csv` | **PASS** |

---

## 5. BẢNG TỔNG HỢP CASE ID → LOẠI TẤN CÔNG → DẤU HIỆU → BIỆN PHÁP

| Case ID | Loại tấn công / Mối đe dọa | Dấu hiệu nhận diện | Biện pháp phòng tránh & Ứng phó |
| :---: | :--- | :--- | :--- |
| **TH1** | **Mối đe dọa & Lỗi quản trị** (Baseline) | Thiếu chính sách bảo mật tập trung; cấu hình sai lệch; thao tác nhầm; sự cố phần cứng/môi trường. | Ban hành chính sách quản lý tài sản, sao lưu định kỳ (backup), cập nhật bản vá và thiết lập baseline an toàn. |
| **TH2** | **Mã độc & Né tránh Antivirus** (Malware / EICAR) | Xuất hiện chuỗi mã độc EICAR; Defender kích hoạt cảnh báo, ghi log Event ID 1116 và cách ly tệp. | Bật Real-time Protection, Cloud-delivered Protection; quét định kỳ bằng chữ ký & phân tích hành vi (Heuristic). |
| **TH3** | **Tấn công Mật khẩu & Keylogging** (Credential Attack) | Hàng loạt sự kiện đăng nhập thất bại (Event 4625); mật khẩu bị lộ dù có độ dài và độ phức tạp cao. | Cấu hình khóa tài khoản khi sai nhiều lần (Account Lockout); dùng MFA/FIDO2; đổi mật khẩu khẩn cấp (Rotation). |
| **TH4** | **Backdoor & Duy trì hiện diện** (Persistence / Listener) | Xuất hiện khóa Registry Run (`LAB3_Run_Demo`), Scheduled Task lạ; tiến trình `python.exe` mở cổng lắng nghe 8080. | Giám sát endpoint bằng Sysmon/EDR; kiểm tra điểm khởi động bằng Autoruns; chặn cổng lạ qua Windows Firewall. |
| **TH5** | **Nghe lén & Giả mạo dữ liệu** (Sniffing / Plaintext) | Bắt được dữ liệu nhạy cảm dạng văn bản rõ qua HTTP 8080; traffic HTTPS 443 bị mã hóa hoàn toàn. | Bắt buộc sử dụng HTTPS/TLS; kích hoạt header HSTS chống hạ cấp giao thức; sử dụng VPN trên mạng công cộng. |
| **TH6** | **Từ chối dịch vụ & Tràn hộp thư** (DoS / DDoS / Bombing) | CPU tăng vọt, phản hồi trễ/timeout khi bị tải dồn dập; lưu lượng đổ về từ nhiều IP botnet hoặc lượng lớn email rác. | Cấu hình giới hạn tần suất (Rate Limiting); dùng dịch vụ chống DDoS (CDN, WAF); cấu hình bộ ba SPF, DKIM, DMARC. |
| **TH7** | **Lừa đảo phi kỹ thuật** (Social Engineering / Phishing) | Email mạo danh IT Admin, tạo tâm lý khẩn cấp (dọa khóa tài khoản 24h), chứa link giả mạo cổng đăng nhập nội bộ. | Dùng Secure Email Gateway (SEG); đào tạo nhận thức nhân viên; triển khai xác thực FIDO2 chống website giả mạo. |
| **TH8** | **Khôi phục & Làm sạch hệ thống** (Incident Recovery) | Cổng 8080 đóng, tiến trình lạ bị hủy, khóa Run bị xóa, tài khoản `lab3user` bị vô hiệu hóa (`Enabled: False`). | Thực hiện quy trình Response chuẩn: Containment (cô lập) → Eradication (làm sạch) → Recovery (khôi phục) → Verification. |

---

## 6. CÁC VẤN ĐỀ/LỖI GẶP PHẢI VÀ BIỆN PHÁP KHẮC PHỤC (TROUBLESHOOTING)

1. **Lỗi chuỗi EICAR không kích hoạt cảnh báo Defender (TH2):**
   * *Nguyên nhân:* Quá trình gõ/copy chuỗi text bị biến dạng ký tự (`X5!P%@@[` thay vì `X5O!P%@AP[`), khiến chữ ký hash không khớp với signature chuẩn của antivirus.
   * *Khắc phục:* Viết script ép mảng byte chuẩn ASCII trực tiếp qua `[System.IO.File]::WriteAllBytes`, sau đó chạy lệnh quét cưỡng bức `MpCmdRun.exe -Scan -ScanType 3`. Windows Defender lập tức phát hiện và cách ly mối đe dọa (`Threat quarantined - Severe`).

2. **Lỗi `InvalidPasswordException` khi tạo người dùng `lab3user` (TH3):**
   * *Nguyên nhân:* Môi trường Windows Server mặc định kích hoạt chính sách bảo mật mật khẩu phức tạp (**Password Complexity Policy**), mật khẩu nhập vào ban đầu không đủ độ phức tạp.
   * *Khắc phục:* Đặt mật khẩu phức tạp đáp ứng tiêu chuẩn gồm chữ hoa, chữ thường, số và ký tự đặc biệt (ví dụ: `Luc2182004#` và đổi sang `Tei2182004#`).

3. **Lỗi `RUNAS ERROR: Unable to acquire user password` (TH3):**
   * *Nguyên nhân:* Lệnh `runas.exe` cổ điển bị xung đột luồng Standard Input khi chạy trong Windows Terminal / PowerShell hiện đại của Windows Server; hoặc dịch vụ `Secondary Logon (seclogon)` chưa bật.
   * *Khắc phục:* Kích hoạt dịch vụ `seclogon` và thay thế việc đăng nhập qua cmdlet hiện đại `Get-Credential` kết hợp `Start-Process -Credential` để tạo chính xác các sự kiện xác thực Audit Failure (Event ID 4625) trong Security Log.

4. **Lỗi không tìm thấy file cấu hình Sysmon (`Failed to open xml configuration`) (TH4):**
   * *Nguyên nhân:* File `sysmon-lab.xml` khi giải nén nằm tại thư mục gốc `C:\LAB3_Threats_Assets\lab3_assets\sysmon-lab.xml` thay vì đường dẫn con khác.
   * *Khắc phục:* Sử dụng `Get-ChildItem` quét vị trí chính xác của tệp XML và truyền đường dẫn đầy đủ vào cờ `-i` của `Sysmon64.exe`.

5. **Mục `LAB3_Run_Demo` không hiển thị trong tab Logon của Autoruns (TH4):**
   * *Nguyên nhân:* Tính năng ẩn các mục mặc định của hệ thống (`Hide Microsoft Entries` / `Hide Windows Entries`) trong Autoruns vô tình ẩn file `notepad.exe`.
   * *Khắc phục:* Ghi đồng thời khóa Run vào cả `HKCU` lẫn `HKLM`, vào menu **Options** của Autoruns bỏ chọn 2 bộ lọc ẩn trên, sau đó nhấn **F5** để làm mới danh sách.

6. **Lỗi `Access Denied` khi đưa file cài đặt Npcap vào máy ảo (TH5):**
   * *Nguyên nhân:* Cố gắng dán tệp trực tiếp vào thư mục được bảo vệ nghiêm ngặt của hệ thống (`C:\Program Files`) mà không thông qua cơ chế UAC Administrator.
   * *Khắc phục:* Đưa file cài đặt `npcap-1.88.exe` vào thư mục cá nhân `$env:USERPROFILE\Downloads`, sau đó dùng PowerShell khởi chạy với tham số `-Verb RunAs` và tích chọn tính năng hỗ trợ Loopback Adapter.

7. **Lỗi PowerShell hiểu nhầm đường dẫn thư mục là Command (TH8):**
   * *Nguyên nhân:* Gõ trực tiếp đường dẫn `C:\LAB3\Evidence\` vào dấu nhắc lệnh PowerShell khiến hệ thống báo lỗi `CommandNotFoundException` vì hiểu nhầm là một tệp thực thi.
   * *Khắc phục:* Sử dụng cmdlet điều hướng thư mục chuẩn `Set-Location "C:\LAB3\Evidence"` (hoặc `cd`) để truy cập thư mục trước khi thực hiện xuất mã băm SHA-256.

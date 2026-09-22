
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
* **Chế độ mạng chính (Isolation):** VMware Host-only Network (Chỉ chuyển sang NAT có kiểm soát khi cần tải công cụ hoặc thử nghiệm bắt gói ngoài mạng)
* **Ngôn ngữ thực thi & Môi trường runtime:**
  * Windows PowerShell 5.1 / PowerShell 7 (Run as Administrator)
  * Python 3.14.x
* **Bộ công cụ Sysinternals & Giám sát:**
  * Microsoft Sysmon v15.22 (Tích hợp schema cấu hình `sysmon-lab.xml`)
  * Sysinternals Autoruns v14.3
  * Sysinternals Process Explorer v17.06
* **Công cụ phân tích mạng:**
  * Wireshark v4.6.8
  * Npcap Driver v1.88 (Bắt buộc bật tính năng Loopback Traffic Capture)
* **Thư mục tài nguyên & Lưu trữ bằng chứng:**
  * Thư mục dự án/assets: `C:\LAB3_Threats_Assets\lab3_assets\`
  * Thư mục lưu bằng chứng kiểm thử: `C:\LAB3\Evidence\`

---

## 3. CÁCH DỰNG VÀ CHUẨN BỊ MÔI TRƯỜNG (SETUP INSTRUCTIONS)
1. **Thiết lập thư mục làm việc:** Tạo cấu trúc thư mục làm việc chuẩn `C:\LAB3\Evidence`, `C:\LAB3\Tools`, và giải nén gói bài tập vào `C:\LAB3_Threats_Assets\lab3_assets\`.
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
| **TH5** | **Sniffing & MITM: So sánh HTTP vs HTTPS** | Sử dụng Wireshark bắt gói tin Loopback để phát hiện dữ liệu bản rõ (plaintext - chuỗi `TRAINING_ONLY`) qua HTTP port 8080; tạm chuyển NAT bắt gói tin cổng 443 để chứng minh dữ liệu được mã hóa bảo vệ (Encrypted Application Data) qua TLS. | `H9_HTTP_Plaintext.png`, `H10_TLS_443.png` | **PASS** |
| **TH6** | **DoS / DDoS / Mail Bombing** | Chưa thực hiện (Hết thời gian làm bài). | N/A | **INCOMPLETE** |

---

## 5. CÁC VẤN ĐỀ/LỖI GẶP PHẢI VÀ BIỆN PHÁP KHẮC PHỤC (TROUBLESHOOTING)

Trong quá trình thực hành, một số sự cố kỹ thuật đã phát sinh và được xử lý thành công:

1. **Lỗi chuỗi EICAR không kích hoạt cảnh báo Defender (TH2):**
   * *Nguyên nhân:* Quá trình gõ/copy chuỗi text bị biến dạng ký tự (`X5!P%@@[` thay vì `X5O!P%@AP[`), khiến chữ ký hash không khớp với signature chuẩn của antivirus.
   * *Khắc phục:* Viết script ép mảng byte chuẩn ASCII trực tiếp qua `[System.IO.File]::WriteAllBytes`, sau đó chạy lệnh quét cưỡng bức `MpCmdRun.exe -Scan -ScanType 3`. Windows Defender lập tức phát hiện và cách ly mối đe dọa (`Threat quarantined - Severe`).

2. **Lỗi `InvalidPasswordException` khi tạo người dùng `lab3user` (TH3):**
   * *Nguyên nhân:* Môi trường Windows Server mặc định kích hoạt chính sách bảo mật mật khẩu phức tạp (**Password Complexity Policy**), mật khẩu nhập vào quá ngắn (< 7 ký tự).
   * *Khắc phục:* Đặt mật khẩu phức tạp đáp ứng tiêu chuẩn gồm chữ hoa, chữ thường, số và ký tự đặc biệt (ví dụ: `Lab3@Pass2026!`).

3. **Lỗi `RUNAS ERROR: Unable to acquire user password` (TH3):**
   * *Nguyên nhân:* Lệnh `runas.exe` cổ điển bị xung đột luồng Standard Input khi chạy trong Windows Terminal / PowerShell hiện đại của Windows Server; hoặc dịch vụ `Secondary Logon (seclogon)` chưa bật.
   * *Khắc phục:* Kích hoạt dịch vụ `seclogon` và thay thế việc đăng nhập qua cmdlet hiện đại `Get-Credential` kết hợp `Start-Process -Credential` để tạo chính xác các sự kiện xác thực Audit Failure (Event ID 4625) trong Security Log.

4. **Lỗi không tìm thấy file cấu hình Sysmon (`Failed to open xml configuration`) (TH4):**
   * *Nguyên nhân:* File `sysmon-lab.xml` khi giải nén nằm tại thư mục gốc `C:\LAB3_Threats_Assets\lab3_assets\sysmon-lab.xml` thay vì đường dẫn mặc định trong giáo trình.
   * *Khắc phục:* Sử dụng `Get-ChildItem` quét vị trí chính xác của tệp XML và truyền đường dẫn đầy đủ vào cờ `-i` của Sysmon64.exe.

5. **Mục `LAB3_Run_Demo` không hiển thị trong tab Logon của Autoruns (TH4):**
   * *Nguyên nhân:* Tính năng ẩn các mục mặc định của hệ thống (`Hide Microsoft Entries` / `Hide Windows Entries`) trong Autoruns vô tình ẩn file `notepad.exe`.
   * *Khắc phục:* Ghi đồng thời khóa Run vào cả `HKCU` lẫn `HKLM`, vào menu **Options** của Autoruns bỏ chọn 2 bộ lọc ẩn trên, sau đó nhấn **F5** để làm mới danh sách.

6. **Lỗi `Access Denied` khi đưa file cài đặt Npcap vào máy ảo (TH5):**
   * *Nguyên nhân:* Cố gắng dán tệp trực tiếp vào thư mục được bảo vệ nghiêm ngặt của hệ thống (`C:\Program Files`) mà không thông qua cơ chế UAC Administrator.
   * *Khắc phục:* Đưa file cài đặt `npcap-1.88.exe` vào thư mục cá nhân `$env:USERPROFILE\Downloads`, sau đó dùng PowerShell khởi chạy với tham số `-Verb RunAs` và tích chọn tính năng hỗ trợ Loopback Adapter.

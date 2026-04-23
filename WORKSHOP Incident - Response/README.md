# **Workshop:** Xử lý & Ứng cứu sự cố tấn công mã hóa dữ liệu trên hệ thống CNTT
### **Organizer:** CLB Bảo mật F-SEC - Đại học FPT Cần Thơ
### **Scenario:** Ứng cứu sự cố tấn công vào hệ thống ảo hóa và sao lưu qua lỗ hổng VEEAM BACKUP
---

> [!IMPORTANT]
> Nội dung được thực hiện trong môi trường Lab cho mục đích giáo dục và nghiên cứu học thuật. Các địa chỉ IP, tên miền và các thông tin trong bài viết không liên quan đến bất kỳ cá nhân, tổ chức thực tế nào.

---

## **I. Execute Summary: (Tóm tắt điều hành)**
- **Loại sự cố:** Tấn công mã hóa dữ liệu (Ransomware) kết hợp tống tiền (Double Extortion) và khai thác tài nguyên trái phép (Cryptojacking).
- **Mã độc nhận diện:**
    + **Ransomware:** Lockbit 3.0 (nhận diện qua file thực thi LB3.exe và Tox ID của LocBitSupp)
    + **Backdoor/C2 (Command & Control):** Cobalt Strike (beacon được cài đặt qua Task Scheduler để duy trì sự hiện diện)
    + **Mining:** Sử dụng NSSM (Non-Sucking Service Manager) để chạy ẩn dưới tên `svchost`.
- **Mục tiêu tấn công:**
    + **Hạ tầng ảo hóa:** VMESXi (chiếm quyền root, tắt VM, đổi mật khẩu)
    + **Hệ thống sao lưu:** VEEAM BACKUP (vô hiệu hóa khả năng phục hồi)
    + **Dữ liệu nghiệp vụ:** OracleDB và RabbitMQ (dump database trước khi mã hóa)

## **II. Incident Timeline: (Dòng thời gian sự cố)**
| Thời gian (UTC+7) | Hành động của kẻ tấn công | Phương pháp | Minh chứng |
| :--- | :---: | :--- |:--- |
| 2024-08-16 | Bắt đầu rà quét (Reconnaissance) | Check `/var/log/hostd.log` | Keyword: `UnimplementedRequestHandler` |
| *2024-08-16 15:55* | *Tạo account sysad* | *Sử dụng Hayabusa (/Event Viewer) phân tích Event Logs* | *Event ID 4720* |
| 2024-08-18 08:24 | Đổi mật khẩu root ESXi | Check `/var/log/hostd.log` | Keyword: `Password was changed` |
| *2024-08-18 15:22* | *Dump dữ liệu Queue* | *Kiểm tra thư mục Public* | *File: msg-0063 trong queue/* |
| *2024-08-18 16:00* | *Thực thi Ransomware LockBit 3.0* | *Amacache và Shimcache* | *Thực thi LB3.exe trong folder Public* |

## **III. Tools & Methodology (Công cụ và Phương pháp)**
- **Ubuntu LiveCD:** Dùng để can thiệp vật lý vào phân vùng hệ thống ESXi và reset mật khẩu root.
- **SSH & grep:** Truy cập từ xa và truy vấn, lọc dữ liệu từ các tệp tin (`hostd.log`, `auth.log`, `syslog.log`).
- **Hayabusa:** Phân tích Event Log, dùng để xác định tấn công Brute-force.
- ...


## **IV. Technical Analysis: (Phân tích kỹ thuật)**
### **1. Recovery & Initial Investigation (Phục hồi quyền truy cập)**
Do máy chủ ESXi bị kẻ tấn công đổi mật khẩu root, cần tiến hành phương pháp **Offline Password Reset**:
- **Kỹ thuật:** Sử dụng môi trường Ubuntu LiveCD để truy cập vào phân vùng hệ thống của ESXi
- **Thực thi:** Mount phân vùng `/dev/sda5`, giải nén các file cấu hình hệ thống `state.tgz` &rarr; `local.tgz`
- **Can thiệp:** Chỉnh sửa file `/etc/shadow`, xóa chuỗi hash mật khẩu của người dùng root để đưa về trạng thái mật khẩu trống.
- **Kết quả:** Đănng nhập thành công vào giao diện quản trị (ESXi Host Client)
### **2. Initial Access & Brute-force (Phân tích dấu vết xâm nhập)**
Dựa trên Event Logs, kẻ tấn công nhắm vào mục tiêu là máy chủ VEEAM (Hostname: WIN-4L207IP3NA9)
- **Hành vi:** Tấn công Brute-force vào giao thức quản trị
- **Kỹ thuật:** Sử dụng công cụ **Hayabusa** phân tích Security Logs, phát hiện 10,223 sự kiện có Event ID 4625 với thông báo "Bad user or pw"
- **Kết quả:** Chiếm quyền thành công và tạc account quản trị giả mạo tên `sysad` lúc 2024-08-16 15:55.
### **3. Lateral Movement & Credential Theft (Di chuyển ngang hàng và Chiếm quyền)**
### **4. Persistence & C2 (Duy trì sự hiện diện và Kết nối C2)**
### **5. Impact - LockBit 3.0 (Giai đoạn thực thi mã độc và Phá hoại)**

## **V. Lessons Learned (Bài học rút ra)**
- **Trong cài đặt, cấu hình:**
   + Cấu hình xác thực bổ sung cho Fortinet.
   + Cấu hình cơ chế fail2ban cho các dịch vụ xác thực
   + Cập nhật bản vá kịp thời
- **Trong khắc phục sự cố và điều tra chiếm quyền điều khiển:**
   + Nhanh chóng cô lập hệ thống
   + Xác định rõ các thành phần bị tấn công
   + Truy vết sâu, phân tích log tỉ mỉ
   + Vá lỗi kịp thời

---
### Acknowledgments
Trân trọng cám ơn **CLB Bảo mật F-SEC - Đại học FPT Cần Thơ** đã tổ chức Workshop và cung cấp môi trường, kịch bản diễn tập thực tế để tôi có cơ hội thực hành kỹ năng ứng cứu sự cố.

---
Author: Dang Lam My Khanh

# **Workshop:** Xử lý & Ứng cứu sự cố tấn công mã hóa dữ liệu trên hệ thống CNTT
### **Organizer:** CLB Bảo mật F-SEC - Đại học FPT Cần Thơ
### **Scenario:** Ứng cứu sự cố tấn công vào hệ thống ảo hóa và sao lưu qua lỗ hổng VEEAM BACKUP

## **I. Execute Summary:**
- **Loại sự cố:** Tấn công mã hóa dữ liệu (Ransomware) kết hợp tống tiền (Double Extortion) và khai thác tài nguyên trái phép (Cryptojacking).
- **Mã độc nhận diện:**
    + **Ransomware:** Lockbit 3.0 (nhận diện qua file thực thi LB3.exe và Tox ID của LocBitSupp)
    + **Backdoor/C2 (Command & Control):** Cobalt Strike (beacon được cài đặt qua Task Scheduler để duy trì sự hiện diện)
    + **Mining:** Sử dụng NSSM (Non-Sucking Service Manager) để chạy ẩn dưới tên svchost.
- **Mục tiêu tấn công:**
    + **Hạ tầng ảo hóa:** VMESXi (chiếm quyền root, tắt VM, đổi mật khẩu)
    + **Hệ thống sao lưu:** VEEAM BACKUP (vô hiệu hóa khả năng phục hồi)
    + **Dữ liệu nghiệp vụ:** OracleDB và RabbitMQ (dump database trước khi mã hóa)

## **II. Incident Timeline:**
| Thời gian (UTC+7) | Hành động của kẻ tấn công | Phương pháp | Minh chứng |
| :--- | :---: | :--- |:--- |
| 2024-08-16 | Bắt đầu rà quét (Reconnaissance) | Check /var/log/hostd.log | Keyword: UnimplementedRequestHandler |

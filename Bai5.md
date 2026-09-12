# BÁO CÁO ĐÁNH GIÁ ĐA PHƯƠNG ÁN KỸ THUẬT (TRADE-OFF) CHO RIKKEICARE MOBILE
**Khóa học:** Phân tích & Thiết kế Hệ thống (IT105)  
**Session:** 03 - Đánh giá Đa phương án Kỹ thuật (Trade-off Matrix) & Chuẩn hóa NFR  
**Vai trò thực hiện:** Lead System Analyst (Lead SA)

---

## PHẦN 1: TÓM TẮT NGUYÊN NHÂN THẤT BẠI CỐT LÕI (TECHNICAL ROOT CAUSES)

1. **Về Hiệu năng & Chịu tải (Performance & Scalability):**  
   Ứng dụng load ảnh X-Quang/MRI trực tiếp từ máy chủ ở định dạng dung lượng gốc (Full Resolution DICOM/RAW) trên kiến trúc xử lý đồng bộ (Synchronous Rendering) và thiếu hạ tầng lưu đệm (CDN/Caching Layer), dẫn đến tràn bộ nhớ RAM thiết bị (Out-Of-Memory) và thắt nút cổ chai kết nối Database khi số lượng người dùng đồng thời vượt quá 150.

2. **Về Bảo mật dữ liệu (Security):**  
   Dữ liệu hình ảnh y tế và thông tin bệnh án truyền tải qua giao thức HTTP không mã hóa (Plaintext Transport) và thiếu mã hóa dữ liệu tĩnh (Data-at-Rest Encryption), làm rò rỉ toàn bộ thông tin nhạy cảm trước các cuộc tấn công Man-in-the-Middle (MitM) và vi phạm chuẩn bảo mật y tế.

3. **Về Tính khả dụng & Giao diện (Usability & Accessibility):**  
   Thiết kế giao diện UI/UX vi phạm các nguyên tắc thiết kế khả dụng cho người cao tuổi (Mobile Accessibility Standard Guidelines - WCAG 2.1 Level AA) khi sử dụng kích thước font chữ tĩnh dưới 12pt, diện tích vùng bấm nút bấm (Touch Target) dưới 32px và đặt nút thao tác nguy hiểm "Hủy phiếu khám" sát các vùng thao tác thường xuyên mà không có bước xác nhận cảnh báo.

---

## PHẦN 2: ĐỀ XUẤT ĐA PHƯƠNG ÁN KỸ THUẬT & BẢNG PHÂN TÍCH TRADE-OFF

### 1. Đề xuất 2 Phương án Kiến trúc Giải pháp Kỹ thuật

- **Phương án 1:** **Kiến trúc Truyền thống (Monolithic Backend + HTTP/REST API + Local File Storage)**  
  Tải ảnh trực tiếp từ máy chủ lưu trữ tệp tin nội bộ qua REST API tiêu chuẩn, mã hóa dữ liệu cơ bản bằng HTTPS/TLS.
- **Phương án 2:** **Kiến trúc Tối ưu Hiện đại (Cloud-Native Microservices + Medical CDN / Dynamic Image Compression + Encrypted Object Storage & TLS 1.3)**  
  Sử dụng dịch vụ nén ảnh động theo phân giải màn hình (Dynamic Image Resizing/Pyramid DICOM), lưu đệm qua mạng phân phối nội dung (Medical CDN), mã hóa dữ liệu lưu trữ bằng AES-256 bits và mã hóa đường truyền bằng giao thức TLS 1.3 với mã hóa đầu cuối (End-to-End Encryption).

---

### 2. Bảng Phân tích Đánh đổi (Trade-off Matrix)

| Tiêu chí so sánh | Phương án 1: Kiến trúc Truyền thống (Monolithic REST API) | Phương án 2: Kiến trúc Tối ưu Hiện đại (Cloud CDN & Image Compression) | Lập luận đánh đổi (Trade-off Analysis) |
| :--- | :--- | :--- | :--- |
| **Thời gian phản hồi truy vấn** | Rất chậm (**8.0s - 45.0s** đối với ảnh MRI/X-Quang dung lượng lớn; sập ứng dụng khi chạm ngưỡng 150 CCU). | Rất nhanh (**0.8s - 1.2s** nhờ nén động theo kích thước màn hình và lưu đệm tại bộ nhớ Edge CDN). | **Đánh đổi giữa Chi phí & Hiệu năng:** Phương án 2 tối ưu thời gian phản hồi gấp ~40 lần và chịu tải cực tốt, nhưng yêu cầu chi phí đầu tư hạ tầng Cloud/CDN và độ phức tạp kỹ thuật cao hơn Phương án 1. |
| **Mức độ an toàn bảo vệ dữ liệu** | Trung bình (chỉ mã hóa HTTPS cơ bản, dữ liệu lưu trữ dạng file tĩnh trên server có rủi ro rò rỉ nội bộ). | Rất cao (Mã hóa toàn vẹn dữ liệu lưu trữ với **AES-256**, mã hóa truyền tải **TLS 1.3** & mã hóa đầu cuối E2EE). | **Đánh đổi giữa Bảo mật & Chi phí vận hành:** Phương án 2 đạt chứng nhận bảo mật y tế tuyệt đối (PCI-DSS / HIPAA compliant), giảm thiểu nguy cơ lộ lọt dữ liệu nhưng làm tăng thời gian xử lý mã hóa/giải mã ở Backend. |

---

### 3. Lập luận Khẳng định Lựa chọn Phương án Tối ưu
*"Mặc dù Phương án 2 (Kiến trúc Tối ưu Cloud CDN & Dynamic Compression) có chi phí đầu tư hạ tầng ban đầu và độ phức tạp phát triển cao hơn, RikkeiCare bắt buộc phải chọn **Phương án 2** vì đây là giải pháp duy nhất giải quyết triệt để rủi ro nghẽn tải/văng ứng dụng, đáp ứng trực tiếp Cam kết chất lượng dịch vụ (SLA tải ảnh $\le 1.5s$, chịu tải $\ge 3,000$ CCU) và tuân thủ các quy định bảo mật dữ liệu y tế quốc tế nghiêm ngặt."*

---

## PHẦN 3: ĐẶC TẢ YÊU CẦU PHI CHỨC NĂNG (NFR MATRIX)

| Nhóm NFR | Chỉ số định lượng mục tiêu | Tiêu chí nghiệm thu |
| :--- | :--- | :--- |
| **Hiệu năng (Performance)** | Thời gian tải ảnh xét nghiệm $\le 1.5s$ | Thử nghiệm tải ảnh MRI 50MB hoàn thành trong 1.2s ở điều kiện mạng 4G tiêu chuẩn. |
| **Bảo mật (Security)** | Mã hóa dữ liệu lưu trữ & truyền tải bằng AES-256 & TLS 1.3 | Không thể giải mã hay đọc trộm thông tin khi bắt gói tin trung gian (MitM Test Passed). |
| **Khả dụng (Usability)** | Cỡ chữ $\ge 16pt$, nút bấm $\ge 48px$ với khoảng đệm an toàn | 95% bệnh nhân $> 60$ tuổi thao tác đặt/hủy phiếu thành công tự lực mà không bấm nhầm. |
| **Chịu tải (Scalability)** | **Khả năng phục vụ đồng thời $\ge 3,000$ người dùng truy cập cùng lúc (Concurrent Users)** | **Hệ thống duy trì hoạt động liên tục với Uptime $\ge 99.9\%$, Tỷ lệ lỗi giao dịch $< 0.1\%$ khi thực hiện kiểm thử chịu tải (Load Test) ở mức 3,000 CCU.** |

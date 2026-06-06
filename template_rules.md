# HƯỚNG DẪN TẠO TEMPLATE LAB TỰ ĐỘNG

## 1. Cấu trúc thư mục bắt buộc:
### Lấy tên thư mục được tôi truyền vào prompt để tạo các thư mục hoặc file sau:
**Lưu ý**: nếu không có tên thư mục được truyền vào hãy hỏi lại để lấy thông tin trước khi thực hiện thao tác
- Lấy tên file html trong thư mục và ghi nhớ để phục vụ cho việc thao tác công việc sau này.
- Tạo 1 thư mục tên là `screenshots` để chứa ảnh minh chứng.
- Tạo 1 file tên là `report.md` nằm ở thư mục gốc.
- Ngoài thư mục và file trên thì hãy đọc mục **What to submit** trong file html trong thư mục mà tôi truyền vào để tự tạo thêm.

## 2. Quy tắc sinh nội dung cho file report.md:
- Thêm tiêu đề chính `# * REPORT` ở đầu file với * là nội dung thẻ title trong file html.
- Comment `<!-- ═══ LABS ═══════════════════════════════ -->` sẽ ghi chú cho việc xuất hiện của các yêu cầu lab sau và `<!-- ── LAB 01 ───────────────────────── -->` sẽ ghi chú thứ tự lab để bạn tạo heading theo format sau `## Lab N - Title` với N là thứ tự lab và Title là tên lab trong thẻ `div` có thuộc tính `class` tên `lab-title`
- Dưới mỗi heading có format `## Lab N - Title` hãy tạo `### Screenshots` và `### In report` (nếu có trong mục **Deliverables** của từng lab).
- Dưới `### Screenshots` tạo các mục `#### Task` với `Task` là tên của nội dung thẻ span của thẻ div có class `deliv-text` với các nội dung được bọc trong `<code>` thì hãy thay bằng \` trong markdown. Tương tự với `### In report`.
- Dưới mỗi đề mục `####` trong `### Screenshots`, tự động chèn sẵn snippet: `![Name](screenshots/labN_M.png)` với Name là tên Task trong `####`, N là thứ tự lab trong heading `## Lab N - Title` và M là thứ tự ảnh theo từng lab (tức nếu ở lab 1 có 3 screenshots thì qua lab 2 ảnh đầu M sẽ là 1 rồi tăng dần, cứ qua lab mới là M reset về 1) để tôi tiện chèn ảnh sau này.
Quy tắc khi dùng Gemini để soạn tài liệu
=======================================

1) Phong cách và mục tiêu
- Viết ngắn gọn, rõ ý, ưu tiên bullet/heading để dễ quét nhanh.
- Tránh biệt ngữ khi không cần; nếu phải dùng, giải thích ngắn gọn.
- Ưu tiên ví dụ thực tế, mã minh họa ngắn, chạy được.
- Giữ giọng điệu trung lập, giải thích lý do khi đề xuất.

2) Cấu trúc đề xuất cho mỗi tài liệu
- Mở đầu: mục tiêu, phạm vi, đối tượng chính đọc.
- Nội dung chính: chia mục nhỏ; mỗi mục có tóm tắt 1–2 dòng trước khi đi sâu.
- Ví dụ và lệnh: đặt trong khối mã; ghi rõ bối cảnh (OS, tool, version).
- Lưu ý/anti-pattern: liệt kê rủi ro, lỗi hay gặp, cách tránh.
- Kết thúc: bước kiểm tra nhanh (checklist), link tham khảo nếu có.

3) Quy tắc trình bày
- Dùng Markdown chuẩn; H2/H3 cho tiêu đề; tránh lồng quá sâu (>3 mức).
- Danh sách: dùng dấu “-”; tránh câu dài; mỗi bullet một ý.
- Mã: thêm ngôn ngữ cho fence (```bash, ```json, ```yaml, ...).
- Bảng: chỉ khi cần so sánh ngắn; tối đa ~5 cột.
- Link: ghi tên mô tả thay vì URL trần khi có thể.

4) Khi viết hướng dẫn thao tác
- Nêu yêu cầu trước (quyền, biến môi trường, phiên bản).
- Bước từng dòng: mỗi bước có kết quả mong đợi; nếu cần lặp, nêu điều kiện dừng.
- Với lệnh phá hủy/nhạy cảm: đặt cảnh báo rõ, kèm thay thế an toàn hoặc dry-run.
- Sau mỗi cụm bước, thêm phần kiểm tra (verify) đơn giản.

5) Khi viết tài liệu kiến trúc/thiết kế
- Bối cảnh: vấn đề gì, ràng buộc nào (hiệu năng, chi phí, bảo mật).
- Phương án: 2–3 lựa chọn, ưu/nhược điểm, tiêu chí chọn.
- Quyết định: lý do chọn, trade-off đã chấp nhận, rủi ro còn lại.
- Lộ trình: bước triển khai, đo lường, rollback/mitigation.

6) Mã minh họa
- Giữ tối thiểu nhưng đầy đủ để chạy (kèm lệnh chạy thử nếu cần).
- Chỉ thêm bình luận khi không tự rõ; tránh thừa chữ.
- Nếu cần dữ liệu giả: chỉ tạo nhỏ gọn, dễ hiểu.

7) Kiểm tra chất lượng tài liệu
- Đọc lại để loại bỏ mơ hồ (“có thể”, “nên” không kèm điều kiện).
- Kiểm tra lệnh/dường dẫn: tránh placeholder chưa thay.
- Đảm bảo nhất quán thuật ngữ, đơn vị, format ngày/giờ.
- Thêm checklist cuối: “Đã thử?”, “Đã backup?”, “Đã đo?”, “Đã rollback plan?”.

8) Bảo mật và tuân thủ
- Không chèn khóa, token, dữ liệu nhạy cảm; dùng placeholder rõ ràng.
- Che thông tin nội bộ không cần thiết; tối thiểu quyền trong ví dụ.
- Nếu thao tác liên quan dữ liệu thật: nhắc sao lưu/ẩn danh hóa.

9) Khi thiếu thông tin
- Nêu giả định rõ ràng; liệt kê câu hỏi cần làm rõ.
- Đề xuất bước thử nhỏ để xác thực giả định trước khi viết sâu.

10) Cách yêu cầu Gemini hỗ trợ
- Luôn cung cấp: mục tiêu, đối tượng đọc, môi trường (OS/tool/version), ràng buộc (bảo mật, thời gian, chi phí).
- Yêu cầu định dạng đầu ra cụ thể (dàn ý, checklist, playbook, runbook, ADR).
- Nếu cần cập nhật tài liệu hiện có: gửi đoạn liên quan và yêu cầu giữ nguyên phần không được đổi.

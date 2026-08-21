# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

<!--
HƯỚNG DẪN - đọc rồi XÓA TOÀN BỘ các khối chú thích này sau khi điền xong:

  - Giới hạn: KHÔNG QUÁ 1 TRANG A4, tương đương khoảng 450 - 550 từ nội dung.
  - Chỉ điền vào các chỗ ___ và các ô trong bảng. Không thêm mục mới.
  - Viết bằng câu hoàn chỉnh, không gạch đầu dòng cụt lủn.
  - Kiểm tra độ dài sau khi đã xóa hết chú thích:
        wc -w nop-bai/bao-cao.md
    và xem trước bản in bằng cách mở file trên GitHub rồi Ctrl+P / Cmd+P.
-->

| | |
|---|---|
| Họ và tên | Bùi Thái Sơn |
| MSSV | 2A202601126 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/gliderest/Track2-Day21-BuiThaiSon-2A202601126.git |
| Ngày nộp | 21-08-2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 50 | 0.1 | 5 | 0.71 | 0.87 |
| 2 | 50 | 0.05 | 2 | 0.61 | 0.85 |
| 3 | 200 | 0.1 | 5 | 0.71 | 0.88 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ tham số ở lần chạy số 3 được lựa chọn vì duy trì được mức `f1_score` cao nhất (0.71) đồng thời tối ưu hóa tốt nhất `accuracy` lên mức 0.88. Lần chạy có accuracy cao nhất trùng khớp với lần có f1_score cao nhất. Về sự đánh đổi, khi giảm độ sâu cây (`max_depth`) và tốc độ học (`learning_rate`) quá thấp như ở lần chạy 2, mô hình lập tức bị kém khớp (underfitting) khiến F1 tụt thê thảm. Ngược lại, việc giữ nguyên `learning_rate` ở mức 0.1 và tăng mạnh số lượng cây (`n_estimators` lên 200) ở lần 3 đã giúp mô hình học được nhiều quy luật phức tạp hơn, cải thiện độ chính xác tổng thể mà không bị quá khớp.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu bài toán tồn tại sự mất cân bằng lớp rõ rệt với tỷ lệ lớn mẫu thuộc nhóm thu nhập <=50K. Hệ quả là một mô hình ngây thơ luôn dự đoán "thu nhập thấp" vẫn có thể đạt accuracy rất cao nhưng hoàn toàn vô dụng trong thực tiễn. Chỉ số F1 của lớp dương khắc phục nhược điểm này bằng cách đo lường đồng thời độ chính xác (precision) và độ phủ (recall), phản ánh trung thực khả năng phát hiện của mô hình trên lớp thiểu số (thu nhập >50K). KHÔNG dùng `average="weighted"` hay `average="macro"` vì các phương pháp này sẽ bị lớp đa số lấn át trọng số, làm lu mờ hiệu suất thực tế trên lớp thiểu số quan trọng.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Lỗi thực thi systemd service với mã `status=203/EXEC` | Đường dẫn đến tệp thực thi uvicorn hoặc Python của môi trường ảo `.venv` trong file service bị sai | Cấu hình lại đường dẫn tuyệt đối tại `WorkingDirectory` và `ExecStart` trỏ đúng vào thư mục dự án trên EC2. |
| Pipeline CI/CD gọi `curl` cổng 8080 bị lỗi kết nối (Connection refused) | Dịch vụ systemd báo active nhưng ứng dụng FastAPI crash ngay lập tức do lỗi import module `src.main` | Dùng lệnh `journalctl` đọc log, xác định lỗi và điều chỉnh lại cấu trúc import module khởi chạy của Uvicorn. |
| EC2 báo thiếu thư viện khi chạy mã nguồn | Môi trường ảo Python chưa được khởi tạo và cài dependencies từ requirements.txt | SSH trực tiếp vào EC2, tạo môi trường `.venv` và dùng pip cài đặt đầy đủ các thư viện yêu cầu. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.71 | 0.88 |
| Bước 3 (thêm `train_batch2`) | 0.74 | 0.88 |

**Nhận xét:** Khi tăng gấp đôi lượng dữ liệu huấn luyện, chỉ số F1 đã có sự cải thiện nhẹ (từ 0.71 lên 0.74) trong khi Accuracy giữ vững ổn định ở mức 0.88. Điều này cho thấy việc bổ sung thêm dữ liệu giúp mô hình học các đặc trưng của lớp thiểu số (thu nhập cao) tốt hơn một chút, đồng thời minh chứng được toàn bộ quy trình huấn luyện liên tục đã tự động ghi nhận dữ liệu mới thành công.

---

## 5. Phần Bonus Đã Thực Hiện (nếu có)

- [ ] Bonus 1 - Tracking MLflow từ xa với DagsHub: ___
- [ ] Bonus 2 - Điều chỉnh ngưỡng quyết định: ___
- [ ] Bonus 3 - Báo cáo precision / recall tự động: ___
- [ ] Bonus 4 - Hoàn trả về phiên bản trước: ___
- [ ] Bonus 5 - Cảnh báo lệch lạc dữ liệu: ___
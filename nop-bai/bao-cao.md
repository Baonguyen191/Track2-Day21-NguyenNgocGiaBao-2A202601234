# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyen Ngoc Gia Bao |
| MSSV | 2A202601234 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/Baonguyen191/Track2-Day21-NguyenNgocGiaBao-2A202601234 |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ tham số thứ ba có F1 cao nhất (0.7149), cao hơn bộ thứ nhất (0.7109) và rõ rệt hơn bộ thứ hai (0.6051), nên vượt quality gate 0.65. Accuracy cao nhất lại thuộc bộ thứ nhất (0.8780), không trùng với bộ có F1 cao nhất; điều này cho thấy accuracy không đủ để chọn mô hình khi lớp dương bị mất cân bằng. Thử nghiệm cũng cho thấy giảm learning rate cần tăng số estimator để bù lại, nhưng learning rate thấp kết hợp quá ít cây làm F1 giảm mạnh.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Trong dữ liệu Adult, chỉ khoảng 24,8% mẫu thuộc lớp thu nhập cao, còn khoảng 75,2% thuộc lớp thu nhập thấp. Vì vậy mô hình luôn dự đoán “thu nhập thấp” vẫn đạt accuracy khoảng 0,752, dù không phát hiện được bất kỳ người nào có thu nhập cao. Accuracy chỉ phản ánh tỷ lệ dự đoán đúng trên toàn bộ mẫu và bị lớp đa số chi phối. F1 của lớp dương kết hợp precision và recall, nên phản ánh tốt hơn việc mô hình vừa bắt đúng người thu nhập cao vừa hạn chế gán nhầm. Lab dùng `f1_score(y_eval, preds)` mặc định cho target = 1, không dùng `average="weighted"` hoặc `average="macro"`, vì các cách tính đó có thể bị lớp đa số kéo lên và làm quality gate kém ý nghĩa.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| DVC trên Windows báo lỗi quyền khi dùng thư mục cấu hình mặc định. | DVC cố ghi vào thư mục hệ thống `C:\\ProgramData\\iterative`. | Chuyển các thư mục cấu hình/cache DVC về thư mục workspace và tạo lại các file con trỏ `.dvc`. |
| Môi trường Python hệ thống thiếu MLflow. | Lệnh `python` đang trỏ tới Python ngoài `.venv`. | Chạy bằng `.venv\\Scripts\\python.exe` và kiểm tra lại bằng pytest. |
| Kết quả thêm dữ liệu cần được đánh giá trên cùng holdout. | Tập `train_batch2` chỉ được dùng để bổ sung huấn luyện, không dùng làm holdout. | Gộp hai batch để huấn luyện và giữ nguyên `data/holdout.csv` để so sánh công bằng. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Khi bổ sung thêm 22.361 mẫu cùng phân phối, F1 tăng từ 0.7149 lên 0.7354 và accuracy tăng từ 0.8740 lên 0.8820. Kết quả cho thấy dữ liệu mới giúp mô hình tổng quát tốt hơn trên holdout, đồng thời quy trình Bước 3 vẫn cần được xác nhận bằng một commit dữ liệu trên GitHub.

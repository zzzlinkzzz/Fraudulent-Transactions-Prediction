# Phát hiện giao dịch gian lận

# Nguồn dữ liệu:
Dữ liệu gồm 6.35 triệu giao dịch của khách hàng tại ngân hàng. Tên ngân hàng, mã khách hàng, loại tiền tệ đã được ẩn danh.
Các giao dịch đã được đánh dâu là giao dịch gian lận hoặc không.
Các trường dữ liệu gồm:

- step - Khung giờ giao dịch được thực hiện. Thông tin này đã được làm lệch đi so với thời gian thực. Mỗi bước là 1 giờ. Tổng sô bước là 744 (tương ứng 30 ngày).
- type - 5 nhãn: CASH-IN, CASH-OUT, DEBIT, PAYMENT, TRANSFER.
- amount - số tiền giao dịch.
- nameOrig - Mã khách hàng chuyển tiền. Có **6.353.307** mã khách hàng là tài khoản chuyển.
- oldbalanceOrg - số dư tài khoản đi trước giao dịch.
- newbalanceOrig - Số dư tài khoản đi sau giao dịch.
- nameDest - Mã khách hàng nhận tiền. Có **2.722.362** mã khách hàng là tài khoản nhận.
- oldbalanceDest - Số dư tài khoản đến trước giao dịch. Khách hàng không mở tài khoản trong hệ thống kí hiệu M (Merchants) trước mã khách hàng.
- newbalanceDest - Số dư tài khoản đến sau giao dịch. 
- isFraud - Giao dịch được đánh dấu là gian lận. Trong tập dữ liệu này, hành vi gian lận chủ yếu là đối tượng cô gắng thao túng tài khoản của khách hàng, sau đó rút tiền hoặc chuyển khoản đến một tài khoản khác sau đó rút tiền. **Chỉ có 8.213 giao dịch được đánh dấu là gian lận so với 6.35 triệu giao dịch được xem xét**.
- isFlaggedFraud - Đánh dấu giao dịch cố gắng chuyển khoản số lượng lớn trên 200.000 trong 1 lần giao dịch. Chỉ có 16 giao dịch được đánh dấu.

src: https://www.kaggle.com/datasets/vardhansiramdasu/fraudulent-transactions-prediction

# Vấn đề
Từ các dữ liệu được đánh dấu thủ công từ các phản hồi của khách hàng. Mục tiêu là xây dựng công cụ tự động phát hiện các giao dịch có tính chất tương tự với các giao dịch gian lận để có biện pháp phòng ngừa và ngăn chặn trong tương lai.

# Hướng giải quyết

- Do số lượng tương tác lớn (6.35 triệu click) do đó cần sử dụng công cụ khai phá dữ liệu phân tán và song song => lựa chọn Spark
- Dữ liệu dạng bảng => có thể sử dụng SQL trong môi trường Spark => tăng hiệu quả phân tích, giảm thời gian viết code, tận dụng được tốc độ của Spark
- Dự đoán giao dịch có phải gian lận hay không => khai thác đặc trưng dữ liệu theo phiên truy cập, có thể sử dụng các mô hình phân loại nông.
- Do chênh lệch giữa nhãn gian lận và không gian lận có chênh lệch rất lớn (tương ứng 8.213 và 6.354.407) do đó dữ liệu huấn luyện cần được xử lí mất cân bằng.

# Công cụ/ngôn ngữ:
Python, SQL
PySpark
Pandas
Scikit-learn

# Phân tích dữ liệu

|    type|    num|isFraud|fraud_percent|flag|flag_percent|
|--------|-------|-------|-------------|----|------------|
|TRANSFER| 532909|   4097|         0.77|  16|         0.0|
| CASH_IN|1399284|      0|          0.0|   0|         0.0|
|CASH_OUT|2237500|   4116|         0.18|   0|         0.0|
| PAYMENT|2151495|      0|          0.0|   0|         0.0|
|   DEBIT|  41432|      0|          0.0|   0|         0.0|

* Các giao dịch trả hóa đơn, nộp tiền, debit không có đánh dấu gian lận, thực tế các giao dịch gian lận chủ yếu để rút tiền ra. Do đó các giao dịch có nhãn CASH_IN, PAYMENT, DEBIT sẽ bị loai khỏi dữ liệu huấn luyện. Dữ liệu huấn luyện còn 2.770.409 giao dịch.


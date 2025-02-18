# Hướng dẫn chi tiết về cách lập kế hoạch archive dữ liệu
## 1. Xác định mục tiêu của việc archive dữ liệu
- Tại sao cần archive dữ liệu?
    + Giải phóng dung lượng lưu trữ
    + Cải thiện hiệu suất của hệ thống
    + Bảo vệ dữ liệu khỏi mất mát

- Loại dữ liệu nào cần được archive?
    + Dữ liệu cũ
    + Dữ liệu ít được sử dụng
    + Dữ liệu không còn được cần thiết cho hoạt động hàng ngày

- Thời gian lưu trữ dữ liệu đã archive là bao lâu?
    + Vài năm hay vĩnh viễn

- Tần suất truy cập dữ liệu đã archive?
    + Hiến khi truy cập
    + Chỉ khi có yêu cầu đặc biệt

## 2. Phân loại dữ liệu
- Dựa trên mức độ quan trọng: 
    + Dữ liệu nào quan trọng nhất cần được bảo vệ và lưu trữ lâu dài?

- Dựa trên tần suất sử dụng:
    + Dữ liệu nào được sử dụng thường xuyên
    + Dữ liệu nào ít được sử dụng?

- Dựa trên loại dữ liệu:
    + Dữ liệu có cấu trúc (Vd dữ liệu trong cơ sở dữ liệu)
    + Dữ liệu không có cấu trúc (Văn bản, hình ảnh, video)

## 3. Lựa chọn phương pháp archive dữ liệu?
- Lưu trữ đám mây:
    + Sử dụng các dịch vụ lưu trữ đám mây như Amazon S3Gkacier, Google Cloud Storage, Azure Archive Storage
- Lưu trữ ngoại tuyến:
    + Lưu trữ dữ liệu trên các thiết bị lưu trữ ngoại tuyến như băng từ, ổ đĩa, ổ cứng

- Hệ thống quản lý lưu trữ:
    +Sử dụng các hệ thống quản lý lưu trữ chuyên dụng như Veritas Enterprise Vault, IBM Spectrum Archive

- Công cụ cơ sở dữ liệu:
    + Sử dụng các tính năng archive dữ liệu tích hợp trong các hệ quản trị cơ sở dữ liệu như Oracle, SQL Server

## 4 Xác định quy trình archive dữ liệu:
- Thời điểm archive: Lên lịch archive dữ liệu định kỳ : tháng, năm
- Quy trình di chuyển dữ liệu: Xác thức cách thức di chuyển dữ liệu từ hệ thống chính sang hệ thống lưu trữ archive
- Kiểm tra và xác thực dữ liệu: Đảm bảo dữ liệu archive đầy đủ và chính xác

## 5. Lựa chọn công cụ archive dữ liệu:
- Dựa trên phương pháp archive đã chọn: Chọn công cụ phù hợp với phương pháp lưu trữ bạn muốn sử dụng.
- Dựa trên yêu cầu về bảo mật: Đảm bảo công cụ có các tính năng bảo mật để bảo vệ dữ liệu đã archive.
- Dựa trên chi phí: So sánh chi phí của các công cụ khác nhau.

## 6. Xây dựng chính sách archive dữ liệu:
- Chính sách lưu trữ: Xác định thời gian lưu trữ cho từng loại dữ liệu
- Chính sách truy cập: Xác định ai có quyền truy cập vào dữ liệu đã archive và cách thức truy cập.
- Chính sách bảo mật: Đảm bảo dữ liệu đã archive được bảo mật an toàn.

## 7. Thực hiện và kiểm tra:
- Thực hiện archive dữ liệu theo kế hoạch đã lập
- Kiểm tra và xác thực dữ liệu đã archive để đảm bảo tính toàn vẹn.
- Đánh giá hiệu quả của kế hoạch archive và điều chỉnh khi cần thiết.


### Lưu ý:

Bảo mật dữ liệu: Đảm bảo dữ liệu đã archive được bảo mật an toàn khỏi truy cập trái phép.
Sao lưu dữ liệu: Sao lưu dữ liệu đã archive để tránh mất mát.
Tuân thủ quy định: Tuân thủ các quy định về lưu trữ và bảo vệ dữ liệu.

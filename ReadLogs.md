## Hướng dẫn đọc logs & xử lý sự cố dịch vụ

### 1. Các lỗi phổ biến và hướng xử lý

| Dấu hiệu/lỗi xuất hiện trong logs         | Dòng log ví dụ                          | Ý nghĩa                           | Hành động khuyến nghị                |
|------------------------------------------|-----------------------------------------|-----------------------------------|--------------------------------------|
| Connection refused                       | `Error: Unable to connect to DB`        | Không kết nối được tới DB         | Kiểm tra thông số DB, network, secret|
| OOMKilled                                | `Killed process ... Out Of Memory`      | Pod hết RAM, bị kill             | Tăng limit RAM, kiểm tra memory leak |
| Invalid credentials                      | `Auth failed for user`                  | Lỗi xác thực (secret/config sai)  | Kiểm tra lại secret, config          |
| API trả 500                              | `Unhandled exception...`                | Lỗi chưa xử lý trong app         | Xem lại code, kiểm tra input         |
| Quá nhiều request bất thường             | `大量请求来自IP ...`                     | Có thể bị DDOS hoặc quét bot      | Chặn IP, kiểm tra log, cảnh báo Dev  |

### 2. Cách tìm kiếm log nhanh

```bash
kubectl logs <pod> -n <ns> | grep "error"
kubectl logs <pod> -n <ns> | grep "exception"
```
Nếu dùng Grafana/Loki, truy vấn theo query:

```bash
{app="service-name"} |= "error"
```
### 3. Checklist xử lý khi phát hiện lỗi
- Xác định lỗi thuộc nhóm phổ biến hay không
- Thực hiện hướng dẫn ở bảng trên
- Nếu không xử lý được, escalate lên Dev phụ trách
- Ghi log thao tác vào nhật ký sự cố
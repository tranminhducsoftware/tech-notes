## Partition 
## Sử khác biệt của bảng không sử dụng Partition và có
- Bảng không được phân vùng
+ Dữ liệu được lưu trữ: Toàn bộ dữ liệu của bảng được lưu trữ trong một hoặc một vài segment liên tục trên đĩa.
+ Truy vấn: Khi bạn thực hiện truy vấn, Oracle cần quét toàn bộ bảng (full table scan) để tìm kiếm dữ liệu thỏa mãn điều kiện truy vấn.
+ Hiệu năng:
    Bảng nhỏ: Hiệu năng truy vấn có thể chấp nhận được.
    Bảng lớn: Hiệu năng truy vấn kém, thời gian truy vấn lâu.

- Bảng đã được phân vùng
+ Dữ liệu được lưu trữ: Dữ liệu của bảng được chia thành các phần nhỏ hơn (partition) và có thể được lưu trữ trong các tablespace khác nhau.
+ Truy vấn: Khi bạn thực hiện truy vấn, Oracle sử dụng khóa phân vùng (partition key) để xác định các partition liên quan đến truy vấn và chỉ quét các partition này.
+ Hiệu năng:
    Bảng nhỏ: Hiệu năng truy vấn tương đương với bảng không được phân vùng.
    Bảng lớn: Hiệu năng truy vấn tốt hơn đáng kể so với bảng không được phân vùng, thời gian truy vấn nhanh hơn.

- Giải thích chi tiết
Partition pruning: Khi bạn truy vấn bảng đã được phân vùng, Oracle sử dụng cơ chế partition pruning để loại bỏ các partition không liên quan đến truy vấn. Ví dụ: nếu bạn truy vấn dữ liệu trong một khoảng thời gian cụ thể, Oracle sẽ chỉ quét các partition chứa dữ liệu trong khoảng thời gian đó.
Giảm thiểu I/O: Do chỉ quét các partition liên quan, lượng dữ liệu cần đọc từ đĩa (I/O) giảm đáng kể, giúp tăng tốc độ truy vấn.
Tối ưu hóa truy vấn: Oracle có thể sử dụng các chiến lược tối ưu hóa truy vấn khác nhau cho bảng đã được phân vùng, giúp cải thiện hiệu năng.

### 1. Tính trong suốt của Partitioning
Một trong những ưu điểm lớn của partitioning trong Oracle là tính trong suốt của nó đối với các ứng dụng và câu truy vấn. Điều này có nghĩa là bạn không cần phải thay đổi bất kỳ câu query nào khi làm việc với bảng đã được phân vùng. Oracle sẽ tự động xử lý việc truy cập dữ liệu từ các partition phù hợp dựa trên điều kiện truy vấn của bạn.
### 2. Sự khác biệt về hiệu năng
Tuy nhiên, có một sự khác biệt quan trọng giữa hai loại bảng này, đó là hiệu năng truy vấn.
Bảng thông thường: Khi bạn truy vấn một bảng thông thường, Oracle cần quét toàn bộ bảng (full table scan) để tìm kiếm dữ liệu thỏa mãn điều kiện truy vấn.
Bảng Partitioning: Khi bạn truy vấn một bảng đã được phân vùng, Oracle sử dụng cơ chế partition pruning để xác định các partition liên quan đến truy vấn và chỉ quét các partition này. Điều này giúp giảm thiểu lượng dữ liệu cần quét và cải thiện đáng kể hiệu năng truy vấn, đặc biệt là trên các bảng lớn.
### Kết luận
Về cơ bản, câu query bạn sử dụng để truy vấn bảng đã được phân vùng và bảng không được phân vùng là giống nhau. Tuy nhiên, bảng đã được phân vùng mang lại lợi ích về hiệu năng truy vấn, đặc biệt là trên các bảng lớn

### Hiệu năng khi query sai điều kiện:
Khi bạn truy vấn một bảng phân vùng với điều kiện không liên quan đến khóa phân vùng, hiệu năng truy vấn sẽ tương đương với bảng không được phân vùng. Điều này làm mất đi lợi ích về hiệu năng mà partitioning mang lại.

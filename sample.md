1. Partitioning (Phân vùng)
Cách thức: Chia bảng deals history thành các phần nhỏ hơn (partition) dựa trên một tiêu chí nào đó (ví dụ: ngày tạo giao dịch, loại giao dịch). Mỗi partition có thể được lưu trữ riêng biệt.
Ưu điểm:
Cải thiện hiệu suất truy vấn: Khi truy vấn chỉ cần dữ liệu từ một số partition nhất định, Oracle có thể bỏ qua các partition không liên quan.
Quản lý dữ liệu dễ dàng hơn: Việc quản lý các partition nhỏ hơn dễ dàng hơn so với việc quản lý một bảng lớn duy nhất.
Tăng cường khả năng mở rộng: Khi cần mở rộng dung lượng lưu trữ, bạn có thể thêm partition mới vào bảng một cách dễ dàng.
Nhược điểm:
Yêu cầu kiến thức chuyên môn về Oracle để thiết lập và quản lý partition.
Có thể phát sinh chi phí lưu trữ nếu bạn lưu trữ các partition trên các thiết bị lưu trữ khác nhau.
Ví dụ:
SQL

CREATE TABLE deals_history (
    deal_id NUMBER,
    deal_date DATE,
    ...
)
PARTITION BY RANGE (deal_date) (
    PARTITION deals_history_2022 VALUES LESS THAN (TO_DATE('2023-01-01', 'YYYY-MM-DD')),
    PARTITION deals_history_2023 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD')),
    PARTITION deals_history_old VALUES LESS THAN (TO_DATE('2022-01-01', 'YYYY-MM-DD')) TABLESPACE archive_tablespace
);
2. Table Compression (Nén bảng)
Cách thức: Nén dữ liệu trong bảng để giảm dung lượng lưu trữ.
Ưu điểm:
Tiết kiệm không gian lưu trữ.
Cải thiện hiệu suất truy vấn (trong một số trường hợp).
Nhược điểm:
Có thể làm chậm các thao tác INSERT, UPDATE, DELETE.
Yêu cầu CPU nhiều hơn để giải nén dữ liệu khi truy vấn.
Ví dụ:
SQL

ALTER TABLE deals_history
COMPRESS ADVANCED;
3. Data Pump (Công cụ Data Pump)
Cách thức: Sử dụng công cụ Data Pump để xuất dữ liệu từ bảng deals history vào một file dump, sau đó bạn có thể lưu trữ file dump này ở một nơi khác (ví dụ: lưu trữ đám mây, băng từ).
Ưu điểm:
Linh hoạt: Bạn có thể chọn dữ liệu nào cần xuất.
Tiện lợi: Dữ liệu được lưu trữ trong một file duy nhất.
Nhược điểm:
Yêu cầu kiến thức về công cụ Data Pump.
Việc khôi phục dữ liệu có thể mất thời gian.
Ví dụ:
SQL

-- Xuất dữ liệu
expdp user/password DIRECTORY=DATA_PUMP_DIR DUMPFILE=deals_history.dmp TABLES=deals_history QUERY="WHERE deal_date < DATE '2023-01-01'"

-- Nhập dữ liệu (khi cần)
impdp user/password DIRECTORY=DATA_PUMP_DIR DUMPFILE=deals_history.dmp TABLES=deals_history
4. Combination (Kết hợp)
Bạn có thể kết hợp các phương pháp trên để đạt được hiệu quả tối ưu. Ví dụ: bạn có thể sử dụng Partitioning để chia bảng deals history thành các phần nhỏ hơn, sau đó sử dụng Table Compression để giảm dung lượng lưu trữ của mỗi partition, và cuối cùng sử dụng Data Pump để di chuyển các partition cũ sang một hệ thống lưu trữ archive.

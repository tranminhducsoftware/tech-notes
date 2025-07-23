# 🗃️ Partition & Index – So sánh, Lưu ý và Kinh nghiệm tối ưu trong DBMS

## 1. Sự khác biệt giữa bảng không Partition và có Partition

### 🟠 Bảng **không** Partition
- **Lưu trữ:**  
  - Dữ liệu lưu trong một hoặc một vài segment liên tục trên đĩa.
- **Truy vấn:**  
  - Khi query, DB phải quét toàn bộ bảng (full table scan) để tìm dữ liệu.
- **Hiệu năng:**  
  - **Bảng nhỏ:** Truy vấn tạm ổn.  
  - **Bảng lớn:** Truy vấn rất chậm, thiếu hiệu quả.

### 🟢 Bảng **Partitioned**
- **Lưu trữ:**  
  - Dữ liệu chia nhỏ thành các phần (partition) có thể nằm ở nhiều tablespace khác nhau.
- **Truy vấn:**  
  - Oracle dùng **partition key** để chỉ quét các partition liên quan, không phải toàn bảng.
- **Hiệu năng:**  
  - **Bảng nhỏ:** Gần như không khác biệt.
  - **Bảng lớn:** Rất tối ưu, truy vấn nhanh hơn nhiều.

#### 🛠️ **Giải thích kỹ thuật**
- **Partition pruning:** Oracle chỉ quét những partition có khả năng chứa dữ liệu thỏa điều kiện truy vấn.
- **Giảm I/O:** Ít phải đọc dữ liệu hơn, tăng tốc truy vấn.
- **Tối ưu hóa truy vấn:** Oracle có thể áp dụng nhiều chiến lược tối ưu với bảng đã partition.

---

## 2. Tính trong suốt & hiệu năng của Partitioning

- **Tính trong suốt:**  
  Ứng dụng và câu query **không cần sửa** khi chuyển sang bảng partitioned; Oracle tự động xử lý truy cập partition phù hợp.

- **Hiệu năng:**  
  - **Bảng thường:** Truy vấn = full table scan (chậm).
  - **Bảng partitioned:** Partition pruning, chỉ scan partition liên quan (rất nhanh với bảng lớn).
  - **Nếu query không dùng partition key:** Hiệu năng gần như bảng thường (không tận dụng được partition).

---

## 3. Lưu ý khi đánh index

### ✅ Xác định cột cần đánh index
- **Cột thường xuất hiện trong WHERE**
- **Cột thường JOIN giữa các bảng**
- **Cột có tính chọn lọc cao** (nhiều giá trị khác biệt)

### ✅ Chọn loại index
- **B-tree:** Phổ biến nhất, đa mục đích
- **Bitmap:** Cho cột ít giá trị (VD: giới tính, trạng thái)
- **Hash:** Phù hợp so sánh =

### ✅ Số lượng index
- **Không nên tạo quá nhiều index**  
  → Mỗi index làm chậm INSERT, UPDATE, DELETE  
- **Chỉ index cột thực sự cần thiết**

### ✅ Kiểm tra hiệu năng sau khi tạo index
- **EXPLAIN PLAN:** Xem kế hoạch thực thi, check index có được dùng không
- **Theo dõi thời gian thực hiện query** trước/sau khi đánh index

---

## 4. Cách rà soát database để đánh lại index

### 🔍 Xác định các truy vấn chậm
- Dùng công cụ theo dõi hiệu năng (SQL Profiler, AWR, Slow query log, ...)
- Phân tích log DB

### 🔬 Phân tích các truy vấn chậm
- Kiểm tra WHERE (lọc cột nào?)
- Kiểm tra JOIN (kết nối cột nào?)

### 🧩 Đề xuất index
- Dựa trên phân tích truy vấn thực tế
- Cân nhắc tính chọn lọc, tần suất cập nhật của cột

### 🛠 Thực hiện đánh index
- Dùng lệnh `CREATE INDEX ...`
- Kiểm tra lại hiệu năng truy vấn sau khi index

---

## 📌 Kết luận

- **Partitioning:** Rất quan trọng với bảng lớn, giúp tối ưu truy vấn, giảm I/O – nhưng phải query đúng partition key mới hiệu quả.
- **Index:** Cần thiết nhưng phải chọn đúng cột, loại index và kiểm soát số lượng.  
- **Luôn kiểm tra hiệu năng thực tế trước/sau khi thay đổi index/partition.**


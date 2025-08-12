# C# Property & Type Guidelines

## 1. Chọn Setter
| Loại             | Khi dùng                                           | Ví dụ |
|------------------|---------------------------------------------------|-------|
| `get; set;`      | Thuộc tính cần đổi nhiều lần, không ràng buộc logic| `public decimal Total { get; set; }` |
| `get; private set;` | Chỉ thay đổi bên trong class, bảo vệ invariants | `public OrderStatus Status { get; private set; }` |
| `get; init;`     | Chỉ set lúc khởi tạo, bất biến sau đó              | `public required string Name { get; init; }` |

**Nguyên tắc:**  
- **Domain Entity:** `private set` + method để thay đổi.  
- **DTO/Contracts:** `init` (immutable).  
- **ValueObject/Record:** `init` hoặc ctor readonly.

---

## 2. `required`
- **Bắt buộc** gán trước khi kết thúc khởi tạo (ctor hoặc object initializer).
- Dùng khi thuộc tính là bắt buộc để object hợp lệ.

```csharp
public class User
{
    public required string Username { get; init; }
}
```

---

## 3. `default!`
- **Ý nghĩa:**
  - `!` = null-forgiving, tắt cảnh báo nullable.
  - `default` = giá trị mặc định (null với reference type).
- **Khi dùng `default!`:**
  - Khi EF hoặc constructor đảm bảo sẽ set trước khi sử dụng.
  - Khi cần để compiler yên trong auto-property chưa set giá trị.
- **Ví dụ:**
```csharp
public string Username { get; private set; } = default!;
```
- **Cảnh báo:** Tránh lạm dụng, dễ che bug null.

---

## 4. `Guid.Empty`
- **Giá trị:** Tất cả byte = 0 (`00000000-0000-0000-0000-000000000000`).
- **Công dụng:** Thường để biểu thị "chưa có giá trị".
- **Khuyến nghị:**
  - Không dùng làm khóa chính.
  - Luôn validate khác `Guid.Empty` ở boundary.
```csharp
if (id == Guid.Empty) throw new ArgumentException("Id is required");
```

---

## 5. `sealed`
- **Mặc định:** Dùng `sealed` nếu không cần kế thừa → bảo vệ invariants & dễ maintain.
- **Ngoại lệ:** Khi cần kế thừa (EF TPH/TPT, abstract base entity) → bỏ `sealed`.
- **Nguyên tắc vàng:** “Đóng mặc định, chỉ mở khi có lý do thật sự.”

---

## 6. Mẫu áp dụng

### Domain Entity
```csharp
public sealed class Order
{
    public Guid Id { get; private set; } = Guid.NewGuid();
    public string OrderNo { get; private set; } = default!;
    public OrderStatus Status { get; private set; } = OrderStatus.Draft;

    private Order() { }
    public Order(string orderNo)
    {
        if (string.IsNullOrWhiteSpace(orderNo)) throw new ArgumentException();
        OrderNo = orderNo;
    }

    public void Submit()
    {
        if (Status != OrderStatus.Draft) throw new InvalidOperationException();
        Status = OrderStatus.Submitted;
    }
}
```

### DTO/Contract
```csharp
public record CreateOrderRequest
{
    public required string OrderNo { get; init; }
    public string? Note { get; init; }
}
```

### ValueObject
```csharp
public sealed class Money : IEquatable<Money>
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        if (amount < 0) throw new ArgumentOutOfRangeException();
        Currency = currency ?? throw new ArgumentNullException();
        Amount = amount;
    }

    public bool Equals(Money? other) =>
        other is not null && Amount == other.Amount && Currency == other.Currency;
    public override int GetHashCode() => HashCode.Combine(Amount, Currency);
}
```

---

## 7. Cheatsheet
- Cần bất biến sau init? → `init`.
- Thay đổi chỉ qua method? → `private set`.
- Thuộc tính bắt buộc? → `required` + `init` hoặc ctor.
- Tắt warning null do EF/ctor set? → `default!`.
- ID không rỗng? → check `Guid.Empty` hoặc generate ngay.
- Không cần kế thừa? → `sealed`.

---

## 8. Pitfalls
- Dùng `get; set;` lung tung → phá invariants.
- Lạm dụng `default!` → che bug null.
- Lưu `Guid.Empty` như giá trị hợp lệ.

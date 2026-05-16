
### `useCallback` là gì?
Đây là một Hook của React, cho phép bạn **lưu trữ (cache) một định nghĩa hàm** qua các lần re-render. Nó giống như việc bạn nói với React: "Hãy nhớ hàm này, và chỉ tạo ra bản sao mới khi thực sự cần thiết".

### Công thức cơ bản
```javascript
const cachedFn = useCallback(fn, dependencies);
```
*   **`fn`**: Hàm bạn muốn lưu lại. React sẽ trả về (chứ không gọi) hàm này.
*   **`dependencies`**: Mảng các giá trị mà hàm phụ thuộc vào (props, state, biến...). Khi các giá trị này không đổi, React trả về chính hàm đã lưu lần trước.

### Mục đích chính & Ví dụ cốt lõi
Lý do số một để dùng `useCallback` là **tránh re-render không cần thiết cho component con** khi kết hợp với `memo`.

**Vấn đề:** Trong JavaScript, mỗi lần component chạy lại, một hàm định nghĩa bên trong nó (dù giống hệt) sẽ là một **hàm mới** về mặt tham chiếu. Nếu bạn truyền hàm đó xuống component con đã được bọc bởi `memo`, component con sẽ luôn re-render vì props "đã thay đổi".

**Giải pháp:** `useCallback` đảm bảo trả về cùng một tham chiếu hàm.

```javascript
import { useCallback, memo } from 'react';

// Component con được memo hóa
const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  console.log('📦 ShippingForm re-render');
  return <form>...</form>;
});

function ProductPage({ productId, referrer, theme }) {
  // ❌ Không dùng useCallback: Mỗi lần theme đổi, hàm này mới -> ShippingForm re-render vô ích
  // const handleSubmit = (orderDetails) => { ... };

  // ✅ Dùng useCallback: Hàm chỉ được tạo mới khi productId hoặc referrer thay đổi
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', { referrer, orderDetails });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} /> {/* Giờ đây sẽ bỏ qua re-render khi theme thay đổi */}
    </div>
  );
}
```

### Các trường hợp sử dụng chính
Bạn chỉ nên coi `useCallback` là công cụ **tối ưu hiệu suất**, không phải để sửa logic sai.

1.  **Tránh re-render component con**: Khi kết hợp với `memo` như ví dụ trên.
2.  **Ổn định dependency cho Hook khác**: Khi một hàm được dùng làm dependency của `useEffect`, `useMemo`, hay chính `useCallback` khác. Điều này ngăn Effect chạy lại liên tục.
3.  **Tối ưu Custom Hook**: Khi viết Hook tùy chỉnh, nên bọc các hàm trả về bằng `useCallback` để người dùng Hook của bạn có thể tối ưu tiếp.

### Mối quan hệ với `useMemo`
Đây là cách dễ nhất để hiểu `useCallback`:
*   `useMemo` lưu **kết quả** của một hàm.
*   `useCallback` lưu **chính hàm đó**.

Thực tế, `useCallback(fn, deps)` tương đương với `useMemo(() => fn, deps)`.

### Lưu ý quan trọng
*   **Đừng lạm dụng**: Chỉ dùng khi thực sự cần tối ưu. Lạm dụng làm code khó đọc hơn.
*   **Vẫn tạo hàm mới**: `useCallback` không ngăn bạn tạo hàm mới mỗi lần render. Nó chỉ quyết định trả về hàm cũ hay hàm mới cho bạn.
*   **Luôn khai báo đúng dependency**: Nếu thiếu hoặc sai, hàm được cache có thể dùng giá trị cũ, gây lỗi. Hãy dùng ESLint plugin cho React.
*   **Có thể tránh cần dùng**: Các nguyên tắc như giữ logic thuần khiết, tránh nâng state không cần thiết có thể giảm đáng kể nhu cầu memoization thủ công.

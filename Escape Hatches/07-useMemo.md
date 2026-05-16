
### `useMemo` là gì?
Đây là một "móc" (Hook) của React, cho phép bạn **lưu trữ (cache) kết quả của một phép tính** và chỉ tính toán lại khi cần thiết, giúp tối ưu hiệu suất.

### Tại sao cần `useMemo`?
Trong React, mỗi khi component re-render, **toàn bộ code trong component sẽ chạy lại**. Nếu có một phép tính phức tạp (như xử lý mảng lớn) không thay đổi kết quả, việc chạy lại là lãng phí. `useMemo` giúp bạn bỏ qua phép tính đó.

### Cách dùng cơ bản
```javascript
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  // Chỉ tính toán lại visibleTodos khi todos hoặc tab thay đổi
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab), // Hàm tính toán
    [todos, tab]                   // Mảng phụ thuộc
  );
}
```
*   **Hàm tính toán**: Là nơi bạn đặt phép tính nặng.
*   **Mảng phụ thuộc**: Liệt kê các biến mà phép tính phụ thuộc vào. Khi các biến này không đổi, `useMemo` sẽ trả về kết quả cũ đã lưu.

### Ba trường hợp nên dùng `useMemo`
Bạn chỉ nên coi `useMemo` như một công cụ tối ưu, không dùng để đảm bảo logic hoạt động đúng.

1.  **Phép tính chậm**: Tính toán rõ ràng là tốn thời gian và phụ thuộc vào các giá trị ít khi thay đổi.
2.  **Tránh re-render component con không cần thiết**: Khi bạn truyền kết quả tính toán xuống một component con đã được bọc bởi `memo` (memo giúp component con chỉ re-render khi props thay đổi). `useMemo` đảm bảo bạn truyền đi *cùng một tham chiếu* nếu dữ liệu không đổi.
3.  **Ổn định dependency cho Hook khác**: Khi kết quả tính toán là một object/array và được dùng làm dependency của `useEffect` hoặc `useMemo` khác. Điều này ngăn Effect chạy lại liên tục.

### Khi nào không cần `useMemo`?
*   Đừng lạm dụng cho mọi thứ. Code sẽ khó đọc hơn.
*   Với các phép tính đơn giản (cộng trừ, nối chuỗi ngắn), việc tạo ra chúng còn nhanh hơn là để React so sánh dependency.

### Lời khuyên quan trọng
Hãy ưu tiên **giải quyết vấn đề gốc rễ** (như chia nhỏ component, đặt state đúng chỗ) trước khi nghĩ đến việc dùng `useMemo` để tối ưu. Một ứng dụng có cấu trúc tốt thường cần rất ít memoization thủ công.

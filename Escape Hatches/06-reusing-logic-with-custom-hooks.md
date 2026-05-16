
### Custom Hooks là gì?
Custom Hooks là các hàm JavaScript do bạn tự tạo để **tái sử dụng logic có trạng thái (stateful logic)** giữa nhiều component React. Chúng cho phép bạn tách biệt phần "làm thế nào" (logic đồng bộ với hệ thống bên ngoài, xử lý sự kiện...) ra khỏi phần "mong muốn làm gì" trong component.

### Quy tắc đặt tên và tạo Hook
*   **Tên phải bắt đầu bằng `use`**: Theo sau là một chữ cái viết hoa (ví dụ: `useOnlineStatus`, `useFormInput`). Quy ước này giúp React và các công cụ như linter nhận biết hàm đó có chứa các Hook khác bên trong và tuân thủ các quy tắc của Hook.
*   **Chỉ dùng tiền tố `use` khi cần**: Nếu hàm của bạn không gọi bất kỳ Hook nào bên trong, đừng dùng tiền tố `use`. Hãy viết nó như một hàm thông thường (ví dụ: `getSorted()` thay vì `useSorted()`).

### Cơ chế hoạt động chính
1.  **Chia sẻ logic, không chia sẻ state**: Mỗi lần bạn gọi một Custom Hook trong một component, state và effect bên trong Hook đó là **hoàn toàn độc lập** với các lần gọi khác. Việc hai component cùng dùng `useOnlineStatus` và cùng hiển thị trạng thái "Online" là vì chúng cùng đồng bộ với một giá trị bên ngoài (trạng thái mạng), không phải vì chúng dùng chung một biến state.
2.  **Nhận giá trị phản ứng (reactive values)**: Custom Hook nhận được props và state mới nhất từ component mỗi khi component re-render. Nhờ vậy, logic bên trong Hook luôn phản ứng chính xác với các thay đổi.
3.  **Xử lý Event Handler**: Khi truyền một event handler vào Custom Hook, bạn nên gói nó trong `useEffectEvent` để tránh việc Effect bên trong Hook bị kích hoạt lại không cần thiết chỉ vì handler thay đổi.

### Khi nào nên và không nên dùng Custom Hook
*   **Nên dùng**:
    *   Khi bạn thấy có sự trùng lặp logic `useEffect` giữa các component, đặc biệt là logic dùng để đồng bộ với hệ thống bên ngoài (fetch dữ liệu, kết nối WebSocket, theo dõi trạng thái trình duyệt...).
    *   Khi bạn muốn làm cho dòng dữ liệu trong component trở nên rõ ràng, khai báo hơn (ví dụ: `const data = useData(url)`).
    *   Để dễ dàng nâng cấp code trong tương lai khi React cung cấp API chuyên biệt hơn.

*   **Hạn chế**:
    *   Không cần thiết tạo Hook cho mọi đoạn code trùng lặp nhỏ (ví dụ: chỉ gói một lệnh `useState`).
    *   Tránh tạo các Hook "vòng đời" (lifecycle) như `useMount(fn)`. Chúng không phù hợp với mô hình React và có thể gây ra lỗi khó phát hiện. Thay vào đó, hãy tập trung vào mục đích cụ thể như `useChatRoom(options)`.

### Lợi ích chính
*   **Tách biệt mối quan tâm**: Component trở nên sạch sẽ, tập trung vào việc hiển thị giao diện dựa trên dữ liệu từ Hook.
*   **Dễ đọc và khai báo**: Mã nguồn thể hiện rõ ý định muốn làm gì (ví dụ: "lấy trạng thái online", "kết nối phòng chat").
*   **Linh hoạt trong cấu trúc**: Bạn có quyền quyết định cách tổ chức, có thể tạo các lớp JavaScript để chứa logic phức tạp và để Effect của bạn đơn giản chỉ cần giao tiếp với lớp đó.

### Tóm tắt
Custom Hooks là một công cụ mạnh mẽ để đóng gói và tái sử dụng logic. Chúng giúp bạn xây dựng các component rõ ràng, dễ bảo trì hơn bằng cách ẩn đi các chi tiết triển khai phức tạp đằng sau một giao diện hàm đơn giản, có tên bắt đầu bằng `use`.
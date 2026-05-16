
### 1. Phân biệt giữa Trình xử lý sự kiện và Effect
Đây là bước đầu tiên để quyết định nên đặt logic của bạn ở đâu.

| Tiêu chí | **Trình xử lý sự kiện (Event Handlers)** | **Hiệu ứng (Effects)** |
| :--- | :--- | :--- |
| **Khi nào chạy?** | Chạy để phản hồi một **tương tác cụ thể** của người dùng (ví dụ: nhấp chuột). | Chạy **tự động** bất cứ khi nào cần để **đồng bộ hóa** với một giá trị bên ngoài. |
| **Tính phản ứng (Reactive)** | **Không phản ứng**. Logic bên trong không tự chạy lại khi giá trị nó đọc thay đổi. Nó chỉ chạy khi người dùng thực hiện lại tương tác. | **Có phản ứng**. Logic bên trong sẽ chạy lại nếu bất kỳ giá trị phản ứng (reactive value) nào mà nó đọc bị thay đổi qua các lần render. |
| **Ví dụ** | Gửi tin nhắn khi nhấn nút "Gửi". | Kết nối đến một phòng chat khi `roomId` thay đổi. |

### 2. Vấn đề với Logic Hỗn hợp
Khó khăn nảy sinh khi bạn muốn kết hợp logic phản ứng và không phản ứng trong cùng một Effect. Ví dụ: Bạn muốn kết nối lại phòng chat **chỉ khi** `roomId` thay đổi (phản ứng), nhưng khi kết nối thành công, bạn muốn hiển thị thông báo với `theme` hiện tại (không phản ứng). Nếu thêm `theme` vào dependency của Effect, kết nối sẽ bị ngắt và tạo lại mỗi khi theme thay đổi, đây là điều không mong muốn.

### 3. Giải pháp: Trích xuất Logic Không Phản ứng với `useEffectEvent`
Hook `useEffectEvent` giúp bạn tách phần logic không phản ứng ra khỏi Effect phản ứng.

*   **Định nghĩa**: `useEffectEvent` tạo ra một "Sự kiện Hiệu ứng" (Effect Event) – một phần của logic Effect nhưng hoạt động giống như một trình xử lý sự kiện. Logic bên trong nó **không phản ứng** và luôn nhìn thấy giá trị mới nhất của props và state.
*   **Cách dùng**:
    1.  Bọc logic không phản ứng (ví dụ: hiển thị thông báo) vào trong `useEffectEvent`.
    2.  Gọi hàm Effect Event đó từ bên trong `useEffect` của bạn.
    3.  Loại bỏ các dependency không còn cần thiết cho tính phản ứng ra khỏi mảng dependency của `useEffect`.

    ```javascript
    function ChatRoom({ roomId, theme }) {
      // 1. Tách logic không phản ứng (dùng theme) vào Effect Event
      const onConnected = useEffectEvent(() => {
        showNotification('Connected!', theme);
      });

      useEffect(() => {
        const connection = createConnection(serverUrl, roomId);
        connection.on('connected', () => {
          // 2. Gọi Effect Event từ trong Effect
          onConnected();
        });
        connection.connect();
        return () => connection.disconnect();
      }, [roomId]); // 3. Chỉ còn 'roomId' là dependency
    }
    ```

### 4. Các Ứng dụng và Lưu ý Quan trọng
*   **Đọc props/state mới nhất**: Effect Event rất hữu ích khi bạn muốn đọc các giá trị mới nhất trong một Effect mà không muốn Effect đó chạy lại khi các giá trị ấy thay đổi (ví dụ: log số lượng mặt hàng trong giỏ khi URL thay đổi, nhưng không log lại khi chỉ số lượng hàng thay đổi).
*   **Giải pháp thay thế tốt hơn việc tắt cảnh báo Linter**: Việc dùng `useEffectEvent` giúp bạn không cần dùng `eslint-disable-next-line` để bỏ qua các dependency, tránh các lỗi khó phát hiện liên quan đến giá trị cũ (stale values).
*   **Giới hạn của Effect Events**:
    *   **Chỉ được gọi từ bên trong Effects.**
    *   **Không được truyền cho component hoặc Hook khác.** Hãy luôn khai báo chúng trực tiếp bên cạnh Effect sử dụng chúng.

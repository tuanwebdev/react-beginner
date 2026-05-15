Dưới đây là phần trình bày kiến thức được tổng hợp từ nguồn bạn cung cấp, giải thích về cách sử dụng **Effects** trong React để đồng bộ hóa với các hệ thống bên ngoài.

### Khái niệm cốt lõi: Effects và Sự kiện (Events)

Trong React, có hai loại logic chính bên trong component:

*   **Mã hiển thị (Rendering code):** Logic thuần túy, nhận props và state, tính toán và trả về JSX. Nó không được chứa tác dụng phụ.
*   **Trình xử lý sự kiện (Event handlers):** Các hàm được kích hoạt bởi hành động cụ thể của người dùng (như nhấp chuột, gõ phím) và chứa các **tác dụng phụ** (ví dụ: gửi yêu cầu HTTP).

**Effects** được dùng cho những tác dụng phụ không xuất phát từ một sự kiện cụ thể mà từ chính việc hiển thị của component, ví dụ: kết nối đến server chat khi một component xuất hiện trên màn hình. Effects chạy ở cuối quá trình commit, sau khi màn hình đã được cập nhật. Đây là thời điểm thích hợp để đồng bộ hóa các component React với một "hệ thống bên ngoài" (như API trình duyệt, thư viện bên thứ ba, mạng lưới).

> **Lưu ý:** Chỉ sử dụng Effects để "bước ra ngoài" React và đồng bộ hóa với hệ thống bên ngoài. Nếu bạn chỉ muốn điều chỉnh state dựa trên state khác, có thể bạn không cần đến Effect.

### Cách viết một Effect (Quy trình 3 bước)

#### Bước 1: Khai báo Effect
Sử dụng Hook `useEffect` và đặt logic tác dụng phụ vào bên trong. Code này sẽ chạy **sau mỗi lần render**. Điều này cho phép bạn trì hoãn một đoạn mã cho đến khi kết quả render đã được hiển thị trên màn hình, tránh làm gián đoạn quá trình tính toán thuần túy của React.
```jsx
import { useEffect } from 'react';

function MyComponent() {
  useEffect(() => {
    // Code ở đây sẽ chạy sau mỗi lần render
  });
  return <div />;
}
```

#### Bước 2: Chỉ định các phần phụ thuộc (Dependencies)
Để tránh việc Effect chạy không cần thiết, bạn cung cấp một mảng các biến phụ thuộc làm đối số thứ hai cho `useEffect`. React sẽ chỉ chạy lại Effect nếu bất kỳ phần phụ thuộc nào trong mảng này thay đổi so với lần render trước đó.
*   `useEffect(fn)`: Chạy sau mỗi lần render.
*   `useEffect(fn, [])`: Chỉ chạy một lần khi component được "gắn" (mount) vào màn hình.
*   `useEffect(fn, [a, b])`: Chạy khi component mount và mỗi khi `a` hoặc `b` thay đổi.

> **Cảnh báo:** Bạn không thể tùy ý chọn dependencies. React sẽ kiểm tra và cảnh báo nếu bạn bỏ sót. Để không phụ thuộc vào một biến, hãy chỉnh sửa logic bên trong Effect.

#### Bước 3: Thêm hàm dọn dẹp (Cleanup) nếu cần
Nếu Effect của bạn thực hiện một hành động cần được hủy bỏ (ví dụ: kết nối, đăng ký sự kiện), hãy trả về một **hàm cleanup** từ bên trong Effect. React sẽ gọi hàm này trước khi Effect chạy lại lần sau và một lần cuối cùng khi component bị gỡ bỏ (unmount).
```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
  // Hàm cleanup
  return () => {
    connection.disconnect();
  };
}, []);
```

### Tại sao Effects chạy hai lần trong Development?

Trong chế độ Strict Mode (phát triển), React cố tình gắn lại (remount) mỗi component một lần ngay sau lần gắn đầu tiên. Điều này nhằm giúp bạn phát hiện các lỗi thiếu hàm cleanup. Việc thấy nhật ký *Kết nối -> Ngắt kết nối -> Kết nối* là hành vi đúng, đảm bảo logic của bạn hoạt động chính xác khi người dùng điều hướng đi và quay lại.

**Cách xử lý:** Thay vì tìm cách ngăn Effect chạy hai lần, hãy tập trung sửa nó để nó hoạt động đúng sau khi được gắn lại. Câu trả lời thường là triển khai hàm cleanup để dừng hoặc hoàn tác những gì Effect đã làm. Quy tắc là người dùng không thể phân biệt được giữa việc Effect chạy một lần (production) với chu trình *thiết lập → dọn dẹp → thiết lập* (development).

### Các mẫu hình phổ biến và Cách xử lý

1.  **Điều khiển widget không phải React, Gửi sự kiện phân tích, Trigger hoạt ảnh:**
    *   **Cách làm:** Viết Effect và hàm cleanup tương ứng để hủy hoặc reset trạng thái về ban đầu. Việc Effect chạy hai lần với cùng giá trị thường không gây ra vấn đề gì khác biệt với người dùng.

2.  **Đăng ký sự kiện:**
    *   **Cách làm:** Hàm cleanup phải hủy đăng ký sự kiện (`removeEventListener`). Điều này đảm bảo tại mỗi thời điểm chỉ có một subscription hoạt động.

3.  **Fetch dữ liệu:**
    *   **Cách làm:** Hàm cleanup không thể hủy yêu cầu mạng đã gửi, nhưng có thể **bỏ qua kết quả trả về** của yêu cầu không còn liên quan (ví dụ: dùng một biến cờ `ignore`). Trong development, bạn sẽ thấy hai yêu cầu mạng, nhưng chỉ kết quả của yêu cầu sau mới được dùng.
    *   **Giải pháp thay thế tốt hơn:** Tự fetch dữ liệu trong Effect có nhiều hạn chế (không chạy trên server, dễ tạo thác nước mạng, không cache). Nên ưu tiên sử dụng cơ chế fetch dữ liệu có sẵn của **framework** hoặc một thư viện **client-side cache** như TanStack Query, SWR.

4.  **Logic KHÔNG phải là Effect:**
    *   **Khởi tạo ứng dụng:** Đặt code chạy một lần khi ứng dụng khởi động **bên ngoài component**.
    *   **Mua hàng:** Đây là kết quả của một tương tác cụ thể (nhấn nút "Mua"), vì vậy hãy đặt logic vào **trình xử lý sự kiện**, không phải Effect.

### Mỗi lần Render đều có Effect riêng

Một điểm quan trọng là mỗi lần render đều "chụp" lại các giá trị props và state của riêng nó, và Effect của lần render đó sẽ chỉ "nhìn thấy" các giá trị đó. React sẽ luôn dọn dẹp Effect của lần render trước trước khi áp dụng Effect cho lần render tiếp theo. Điều này ngăn chặn nhiều lỗi, bao gồm cả race condition.

### Tóm tắt

*   Effects cho phép bạn đồng bộ component với các hệ thống bên ngoài.
*   Effects chạy sau mỗi lần render trừ khi bạn chỉ định mảng dependencies.
*   Mảng dependencies rỗng (`[]`) nghĩa là Effect chỉ chạy khi mount.
*   Strict Mode trong development sẽ chạy setup + cleanup + setup để kiểm tra lỗi.
*   Nếu Effect của bạn gặp lỗi do remount, hãy viết hàm cleanup.
*   React gọi cleanup trước khi Effect chạy lại và khi component unmount.
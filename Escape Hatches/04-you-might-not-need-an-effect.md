
### 🚫 Khi nào bạn không cần Effect?

1.  **Biến đổi dữ liệu để hiển thị:** Nếu có thể tính toán trực tiếp từ props hoặc state hiện có trong lúc render, đừng lưu kết quả vào state và cập nhật bằng Effect. Cách làm này đơn giản và tránh được một lần render không cần thiết.
    *   *Ví dụ:* Thay vì dùng Effect để tạo `fullName` từ `firstName` và `lastName`, hãy tính trực tiếp: `const fullName = firstName + ' ' + lastName;`.

2.  **Xử lý sự kiện người dùng:** Logic được kích hoạt bởi một tương tác cụ thể (như nhấn nút Mua) nên được đặt trong chính trình xử lý sự kiện đó, không phải trong Effect. Điều này giúp xác định rõ ràng nguyên nhân và tránh các lỗi liên quan đến thời điểm chạy.

### 🔄 Các mẫu hình cụ thể và cách thay thế Effect

Bảng dưới đây tóm tắt các tình huống bạn nên tránh dùng Effect và phương án tối ưu hơn:

| Tình huống | Giải pháp thay thế | Giải thích & Ví dụ |
| :--- | :--- | :--- |
| **Tính toán dữ liệu từ props/state** | Tính trực tiếp khi render | Loại bỏ state trung gian và Effect, code nhanh và đơn giản hơn.<br>Thay vì: `useEffect(() => { setFullName(firstName + ' ' + lastName); }, ...)`<br>Hãy dùng: `const fullName = firstName + ' ' + lastName;` |
| **Cache phép tính "đắt"** | Dùng `useMemo` | Chỉ tính toán lại khi dependencies thay đổi, tránh tính lại không cần thiết khi component re-render vì lý do khác.<br>Thay vì: `useEffect(() => { setVisibleTodos(getFilteredTodos(todos, filter)); }, ...)`<br>Hãy dùng: `const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);` |
| **Reset toàn bộ state khi prop thay đổi** | Truyền `key` cho component | Coi mỗi giá trị prop khác nhau là một component hoàn toàn mới, React sẽ tự động reset toàn bộ state bên trong.<br>Thay vì: `useEffect(() => { setComment(''); }, [userId]);`<br>Hãy dùng: `<Profile userId={userId} key={userId} />` |
| **Điều chỉnh một phần state khi prop thay đổi** | Cập nhật state trực tiếp khi render | Hiệu quả hơn Effect vì state được cập nhật ngay trong cùng một lượt render, tránh hiển thị giá trị cũ.<br>Thay vì: `useEffect(() => { setSelection(null); }, [items]);`<br>Hãy cân nhắc: `if (items !== prevItems) { setSelection(null); }` (và nên ưu tiên các giải pháp khác như tính toán selection trực tiếp). |
| **Chia sẻ logic giữa các event handler** | Gọi hàm dùng chung từ các handler | Tránh Effect chạy không đúng lúc (ví dụ: khi refresh trang). Chỉ cần tạo một hàm và gọi nó trong cả hai handler.<br>Thay vì: `useEffect(() => { if (product.isInCart) showNotification(); }, [product]);`<br>Hãy dùng: Tạo hàm `buyProduct()` chứa logic dùng chung và gọi nó trong `handleBuyClick` và `handleCheckoutClick`. |
| **Chuỗi các Effect phụ thuộc lẫn nhau** | Tính toán khi render hoặc gộp vào event handler | Tránh nhiều lần re-render không cần thiết và code dễ hiểu, linh hoạt hơn. Hãy tính toán trực tiếp các giá trị dẫn xuất và xử lý logic cập nhật state tại nơi sự kiện xảy ra. |
| **Thông báo cho component cha về thay đổi state** | Cập nhật state của cả hai component trong cùng một sự kiện | Tận dụng cơ chế batching của React để chỉ re-render một lần.<br>Thay vì: `useEffect(() => { onChange(isOn); }, [isOn, onChange])`<br>Hãy dùng: Trong event handler, gọi `setIsOn(nextIsOn)` và `onChange(nextIsOn)` cùng lúc. |
| **Truyền dữ liệu đã fetch lên component cha** | Fetch dữ liệu ở component cha và truyền xuống | Giữ luồng dữ liệu một chiều từ trên xuống, dễ dự đoán và gỡ lỗi hơn. |
| **Subcribe vào external store** | Dùng `useSyncExternalStore` | Hook chuyên dụng, ít lỗi hơn so với việc tự đồng bộ thủ công bằng Effect.<br>Thay vì: `useEffect(() => { window.addEventListener(...); return () => window.removeEventListener(...); }, []);`<br>Hãy dùng: `const isOnline = useSyncExternalStore(subscribe, () => navigator.onLine);` |
| **Fetch dữ liệu** | Vẫn cần Effect, nhưng phải có **cleanup** và cân nhắc giải pháp của Framework | Đây là trường hợp hợp lệ để dùng Effect. Tuy nhiên, cần có cleanup để tránh race condition. Các framework hiện đại cung cấp cơ chế fetch tích hợp hiệu quả hơn. |

### 💡 Ghi nhớ

- **Chỉ dùng Effect để đồng bộ với hệ thống bên ngoài.** Nếu không có hệ thống ngoài nào tham gia (API trình duyệt, thư viện bên thứ ba, mạng...), rất có thể bạn không cần Effect.
- **Tính toán được lúc render thì đừng dùng Effect.**
- **Logic từ tương tác người dùng thì để trong event handler.**
- **Cần đồng bộ state giữa các component?** Hãy nghĩ đến việc "nâng state lên" (lifting state up).
- **Fetch dữ liệu trong Effect** là một trường hợp ngoại lệ cần làm cẩn thận, luôn nhớ dọn dẹp để tránh race condition và nên dùng giải pháp của framework hoặc custom hook để code sạch hơn.
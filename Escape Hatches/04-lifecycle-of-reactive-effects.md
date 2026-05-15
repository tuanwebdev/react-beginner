
### Tư duy khác biệt: Effect là đồng bộ hóa, không phải vòng đời

Bài viết nhấn mạnh rằng bạn cần thay đổi cách nghĩ về Effect:
*   **Tư duy theo vòng đời component (Mount/Update/Unmount):** Dễ dẫn đến lỗi khi bạn chỉ muốn chạy logic một lần nhưng lại bị chạy lại do dependencies thay đổi.
*   **Tư duy của Effect:** Mục tiêu là **đồng bộ hóa** component với một hệ thống bên ngoài (API, thư viện, mạng...). Effect mô tả *quá trình đồng bộ hóa* tại một thời điểm dựa trên các dependencies hiện tại.

### Vòng đời của một Reactive Effect

Vòng đời của Effect được xác định bởi ba giai đoạn chính, tập trung vào việc bắt đầu và dừng đồng bộ hóa.

#### 1. Bắt đầu Đồng bộ hóa (Setup)
Mỗi khi component được gắn vào cây DOM và các dependencies của Effect khớp với giá trị mới, React sẽ chạy hàm setup của Effect để bắt đầu quá trình đồng bộ hóa.
*   **Lần đầu tiên:** Effect chạy setup sau khi component được render lần đầu và xuất hiện trên màn hình (mount).
*   **Các lần sau:** Effect chạy lại setup sau khi component re-render, nhưng chỉ khi **có ít nhất một dependency trong mảng phụ thuộc thay đổi** so với lần render trước.

#### 2. Kết thúc Đồng bộ hóa (Cleanup)
Trước khi chạy lại hàm setup mới (do dependencies thay đổi) hoặc khi component bị gỡ bỏ (unmount), React sẽ chạy hàm **cleanup** mà bạn đã trả về từ lần setup trước đó. Hàm cleanup có nhiệm vụ dừng hoặc hoàn tác quá trình đồng bộ hóa cũ, tránh rò rỉ bộ nhớ hoặc lỗi logic.

#### 3. Đồng bộ hóa lại (Re-synchronization)
Đây là chu trình cốt lõi: khi dependencies thay đổi, React sẽ:
1.  Gọi hàm **cleanup** của Effect cũ (với giá trị dependencies cũ).
2.  Gọi hàm **setup** của Effect mới (với giá trị dependencies mới).
Chu trình này đảm bảo component luôn được đồng bộ chính xác với trạng thái bên ngoài dựa trên dữ liệu mới nhất.

### Phân biệt rõ các bước trong ví dụ ChatRoom

Bài viết minh họa rất rõ qua component `<ChatRoom roomId={roomId} />` với Effect kết nối đến phòng chat:

```jsx
useEffect(() => {
  const connection = createConnection(roomId);
  connection.connect();
  return () => connection.disconnect();
}, [roomId]);
```

| Giai đoạn | Hành động của Effect | Giải thích |
| :--- | :--- | :--- |
| **Mount (roomId = "general")** | Gọi `createConnection('general')` và `connect()`. | Bắt đầu đồng bộ hóa với phòng "general". |
| **Re-render (cùng roomId "general")** | **Không làm gì.** | Tất cả dependencies (`roomId`) không thay đổi, Effect bị bỏ qua hoàn toàn. |
| **Re-render (roomId = "travel")** | 1. Gọi hàm cleanup cũ: `disconnect()` từ kết nối "general".<br>2. Gọi setup mới: `createConnection('travel')` và `connect()`. | Dừng đồng bộ hóa cũ và bắt đầu đồng bộ hóa mới với phòng "travel". Đây là quá trình đồng bộ hóa lại. |
| **Unmount** | Gọi hàm cleanup cuối cùng: `disconnect()` từ kết nối "travel". | Dừng mọi đồng bộ hóa khi component bị xóa khỏi cây. |

### Mỗi lần Render có một Effect riêng biệt

Một điểm quan trọng khác là mỗi lần render đều "chụp" lại các giá trị props, state và cả Effect của riêng nó. Hàm Effect của lần render nào sẽ chỉ "nhìn thấy" các giá trị tại thời điểm đó. React luôn dọn dẹp Effect của lần render trước trước khi chạy Effect của lần render sau, ngăn chặn các lỗi như race condition.

### Tóm tắt vòng đời Effect

*   **Mục đích:** Đồng bộ hóa component với hệ thống bên ngoài dựa trên dependencies.
*   **Setup:** Chạy khi mount và khi dependencies thay đổi.
*   **Cleanup:** Chạy trước setup mới (khi dependencies thay đổi) và khi unmount.
*   **Re-synchronization:** Chu trình cleanup -> setup mỗi khi dependencies thay đổi.
*   **Tính "chụp ảnh":** Mỗi Effect gắn liền với giá trị của lần render cụ thể, giúp tránh lỗi đồng bộ.

## Giới thiệu
React Router là thư viện định tuyến tiêu chuẩn cho React, giúp đồng bộ giao diện với URL của trình duyệt mà không cần tải lại trang. Thư viện này hỗ trợ xây dựng ứng dụng Single Page Application (SPA) với trải nghiệm điều hướng mượt mà.

---

## 1. Cài đặt

Cài đặt thư viện bằng npm:

```bash
npm install react-router-dom
```

---

## 2. Các thành phần cốt lõi

### `BrowserRouter`

Bao bọc toàn bộ ứng dụng để kích hoạt tính năng định tuyến dựa trên lịch sử trình duyệt.

### `Routes`

Chứa danh sách tất cả các tuyến đường (`Route`) trong ứng dụng.

### `Route`

Ánh xạ một đường dẫn (`path`) cụ thể tới một thành phần giao diện (`element`).

### `Link`

Tạo liên kết điều hướng giữa các trang mà không cần tải lại toàn bộ trang web.

---

## 3. Ví dụ cơ bản (React Router v6)

Ví dụ dưới đây minh họa cách điều hướng giữa 3 trang:

* Trang chủ
* Giới thiệu
* Liên hệ

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

// Định nghĩa các component đơn giản
const Home = () => <h2>Trang chủ</h2>;
const About = () => <h2>Giới thiệu</h2>;
const Contact = () => <h2>Liên hệ</h2>;

export default function App() {
  return (
    <BrowserRouter>
      {/* Thanh điều hướng */}
      <nav style={{ margin: 10, padding: 10 }}>
        <Link to="/">Trang chủ</Link> |{" "}
        <Link to="/about">Giới thiệu</Link> |{" "}
        <Link to="/contact">Liên hệ</Link>
      </nav>

      {/* Cấu hình các tuyến đường */}
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </BrowserRouter>
  );
}
```

---

## 4. Kết quả

Khi chạy ứng dụng:

* Truy cập `/` → hiển thị **Trang chủ**
* Truy cập `/about` → hiển thị **Giới thiệu**
* Truy cập `/contact` → hiển thị **Liên hệ**

Việc chuyển trang diễn ra nhanh chóng mà không cần tải lại toàn bộ website.

---


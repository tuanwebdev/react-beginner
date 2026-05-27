
## Giới thiệu
`useNavigate` là hook trong React Router dùng để điều hướng giữa các trang bằng code thay vì dùng `<Link>`.

---

## Cách import

```jsx
import { useNavigate } from "react-router-dom";
```

---

## Khởi tạo

```jsx
const navigate = useNavigate();
```

---

## Các cách dùng phổ biến

### 1. Điều hướng sang trang khác

```jsx
navigate("/about");
```

---

### 2. Quay lại trang trước

```jsx
navigate(-1);
```

---

### 3. Tiến tới trang tiếp theo trong history

```jsx
navigate(1);
```

---

### 4. Điều hướng sau khi submit form

```jsx
const handleSubmit = () => {
  // xử lý dữ liệu
  navigate("/dashboard");
};
```

---

### 5. Thay thế history hiện tại (`replace`)

Không cho quay lại trang cũ.

```jsx
navigate("/login", { replace: true });
```

---

### 6. Truyền dữ liệu qua `state`

```jsx
navigate("/profile", {
  state: { name: "Nam" }
});
```

Nhận dữ liệu:

```jsx
import { useLocation } from "react-router-dom";

const location = useLocation();

console.log(location.state);
```

---

## Ví dụ đầy đủ

```jsx
import { useNavigate } from "react-router-dom";

function Home() {
  const navigate = useNavigate();

  return (
    <button onClick={() => navigate("/about")}>
      Đi tới trang Giới thiệu
    </button>
  );
}
```


# Conditional Rendering trong React

Nguồn chính: [React Docs - Conditional Rendering](https://react.dev/learn/conditional-rendering?utm_source=chatgpt.com) ([React][1])

---

# Conditional Rendering là gì?

Conditional Rendering = render giao diện theo điều kiện.

React dùng JavaScript bình thường để:

* hiển thị component
* ẩn component
* đổi giao diện theo state/props

Ví dụ:

* Đã login → hiện Dashboard
* Chưa login → hiện Login
* Loading → hiện spinner
* Có dữ liệu → hiện danh sách

---

# Tư duy cực kỳ quan trọng

Trong React:

```jsx id="eqm1"
if (condition) {
  return <A />
}

return <B />
```

nghĩa là:

* condition đúng → render `<A />`
* condition sai → render `<B />`

React chỉ render JSX được return.

---

# Ví dụ cơ bản

```jsx id="eqm2"
function Item({ isPacked, name }) {
  if (isPacked) {
    return <li>{name} ✅</li>;
  }

  return <li>{name}</li>;
}
```

---

# Kết quả

## `isPacked = true`

```html id="eqm3"
<li>Space suit ✅</li>
```

---

## `isPacked = false`

```html id="eqm4"
<li>Space suit</li>
```

([React][1])

---

# 1. Dùng `if`

Đây là cách dễ hiểu nhất.

---

# Ví dụ

```jsx id="eqm5"
function Greeting({ isLogin }) {
  if (isLogin) {
    return <h1>Welcome back</h1>;
  }

  return <h1>Please login</h1>;
}
```

---

# Khi nào nên dùng `if`?

✅ Logic phức tạp
✅ Nhiều điều kiện
✅ JSX dài

---

# 2. Return `null`

Trong React:

```jsx id="eqm6"
return null;
```

=> render NOTHING.

---

# Ví dụ

```jsx id="eqm7"
function Warning({ show }) {
  if (!show) {
    return null;
  }

  return <h1>Warning!</h1>;
}
```

---

# Kết quả

## `show = false`

Không render gì cả.

---

# Đây là kỹ thuật cực kỳ phổ biến

Ví dụ:

* Modal
* Notification
* Tooltip
* Loading
* Error message

---

# 3. Dùng toán tử ba ngôi `? :`

---

# Syntax

```jsx id="eqm8"
condition ? A : B
```

---

# Ví dụ

```jsx id="eqm9"
function Greeting({ isLogin }) {
  return (
    <h1>
      {isLogin ? 'Welcome' : 'Please Login'}
    </h1>
  );
}
```

---

# Kết quả

## `isLogin = true`

```html id="eqm10"
<h1>Welcome</h1>
```

---

## `isLogin = false`

```html id="eqm11"
<h1>Please Login</h1>
```

([React Express][2])

---

# Khi nào nên dùng ternary?

✅ UI ngắn
✅ Có if/else đơn giản

---

# Không nên dùng khi quá dài

❌ Khó đọc:

```jsx id="eqm12"
{
  isLoading
    ? <Loading />
    : isError
      ? <Error />
      : <Dashboard />
}
```

---

# Nên tách ra

```jsx id="eqm13"
if (isLoading) {
  return <Loading />
}

if (isError) {
  return <Error />
}

return <Dashboard />
```

---

# 4. Dùng `&&`

Đây là cách phổ biến nhất để:

✅ render nếu condition đúng

---

# Syntax

```jsx id="eqm14"
condition && <Component />
```

---

# Ví dụ

```jsx id="eqm15"
function App({ isAdmin }) {
  return (
    <div>
      <h1>Dashboard</h1>

      {isAdmin && <button>Delete</button>}
    </div>
  );
}
```

---

# Khi `isAdmin = true`

```html id="eqm16"
<button>Delete</button>
```

---

# Khi `isAdmin = false`

React render nothing.

([React Express][2])

---

# Vì sao `&&` hoạt động?

Trong JavaScript:

```js id="eqm17"
true && "Hello"
```

=> `"Hello"`

---

```js id="eqm18"
false && "Hello"
```

=> `false`

React bỏ qua:

* false
* null
* undefined

---

# Lỗi cực kỳ hay gặp với `&&`

---

# Sai

```jsx id="eqm19"
{
  items.length && <List />
}
```

---

# Khi `items.length = 0`

React render:

```txt id="eqm20"
0
```

---

# Vì:

```js id="eqm21"
0 && <List />
```

=> `0`

---

# Cách đúng

```jsx id="eqm22"
{
  items.length > 0 && <List />
}
```

Hoặc:

```jsx id="eqm23"
{
  !!items.length && <List />
}
```

([React Express][2])

---

# 5. Gán JSX vào biến

Khi logic lớn hơn.

---

# Ví dụ

```jsx id="eqm24"
function Status({ isLoading, error }) {
  let content;

  if (isLoading) {
    content = <p>Loading...</p>;
  } else if (error) {
    content = <p>Error!</p>;
  } else {
    content = <p>Success</p>;
  }

  return <div>{content}</div>;
}
```

---

# Đây là pattern rất phổ biến

Dùng khi:

* nhiều UI branch
* component lớn
* logic phức tạp

([React Express][2])

---

# 6. Early Return Pattern

Đây là cách React dev dùng rất nhiều.

---

# Ví dụ

```jsx id="eqm25"
function Profile({ user }) {
  if (!user) {
    return <Login />;
  }

  return <Dashboard />;
}
```

---

# Ưu điểm

✅ Dễ đọc
✅ Tránh nested code
✅ Code sạch hơn

---

# 7. Conditional Rendering với List

---

# Ví dụ

```jsx id="eqm26"
const items = ['Apple', 'Banana', 'Orange'];

function App() {
  return (
    <div>
      {items.map(item =>
        item.includes('a')
          ? <p key={item}>{item}</p>
          : null
      )}
    </div>
  );
}
```

---

# Ý nghĩa

Chỉ render item chứa chữ `"a"`.

([GeeksforGeeks][3])

---

# 8. Điều kiện với nhiều component

---

# Ví dụ

```jsx id="eqm27"
function App({ isLogin }) {
  return (
    <div>
      {isLogin
        ? <Dashboard />
        : <Login />
      }
    </div>
  );
}
```

---

# 9. Conditional Rendering với CSS

---

# Ví dụ

```jsx id="eqm28"
<button className={isActive ? 'active' : ''}>
  Click
</button>
```

---

# Hoặc

```jsx id="eqm29"
<div style={{
  display: isVisible ? 'block' : 'none'
}}>
```

---

# 10. Dùng function để render condition

---

# Ví dụ

```jsx id="eqm30"
function renderContent(status) {
  if (status === 'loading') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error />;
  }

  return <Success />;
}
```

---

# Component

```jsx id="eqm31"
function App({ status }) {
  return (
    <div>
      {renderContent(status)}
    </div>
  );
}
```

([Reddit][4])

---

# 11. Những giá trị React KHÔNG render

React bỏ qua:

```js id="eqm32"
false
null
undefined
```

---

# Nhưng React VẪN render

```js id="eqm33"
0
''
[]
```

---

# Ví dụ

```jsx id="eqm34"
{0}
```

=> render:

```txt id="eqm35"
0
```

---

# 12. Không được dùng `if` trực tiếp trong JSX

---

# Sai

```jsx id="eqm36"
return (
  <div>
    {
      if (isLogin) {
        <Dashboard />
      }
    }
  </div>
)
```

---

# Vì JSX chỉ chấp nhận expression

Không chấp nhận statement.

---

# Cách đúng

## Dùng ternary

```jsx id="eqm37"
{
  isLogin
    ? <Dashboard />
    : <Login />
}
```

---

## Hoặc `&&`

```jsx id="eqm38"
{
  isLogin && <Dashboard />
}
```

---

## Hoặc early return

```jsx id="eqm39"
if (!isLogin) {
  return <Login />
}

return <Dashboard />
```

---

# 13. So sánh các cách Conditional Rendering

| Cách            | Khi dùng            |
| --------------- | ------------------- |
| `if`            | Logic phức tạp      |
| `return null`   | Ẩn component        |
| `? :`           | if/else ngắn        |
| `&&`            | Chỉ render khi đúng |
| Variable        | UI nhiều branch     |
| Function render | Logic rất lớn       |
| Early return    | Code sạch dễ đọc    |

---

# 14. Best Practices

---

# ✅ Dùng `&&` cho điều kiện đơn giản

```jsx id="eqm40"
{isAdmin && <AdminPanel />}
```

---

# ✅ Dùng ternary cho if/else ngắn

```jsx id="eqm41"
{isLogin ? <Home /> : <Login />}
```

---

# ✅ Dùng early return cho logic lớn

```jsx id="eqm42"
if (loading) return <Loading />;
if (error) return <Error />;
```

---

# ❌ Tránh nested ternary quá nhiều

```jsx id="eqm43"
a ? b ? c : d : e
```

Khó đọc.

([Reddit][5])

---

# Ví dụ tổng hợp

```jsx id="eqm44"
function Dashboard({
  isLogin,
  isAdmin,
  loading
}) {
  if (loading) {
    return <h1>Loading...</h1>;
  }

  if (!isLogin) {
    return <h1>Please Login</h1>;
  }

  return (
    <div>
      <h1>Dashboard</h1>

      {isAdmin && (
        <button>Delete User</button>
      )}
    </div>
  );
}
```

---

# Recap ngắn gọn

| Syntax              | Ý nghĩa               |
| ------------------- | --------------------- |
| `if`                | Render theo điều kiện |
| `return null`       | Không render          |
| `condition ? A : B` | If/else               |
| `condition && A`    | Chỉ render khi đúng   |
| `let content`       | JSX variable          |
| Early return        | Tách logic rõ ràng    |

---

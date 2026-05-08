# JavaScript trong JSX với Dấu Ngoặc Nhọn `{}`

Nguồn chính: [React Docs - JavaScript in JSX with Curly Braces](https://react.dev/learn/javascript-in-jsx-with-curly-braces?utm_source=chatgpt.com) ([react.dev][1])

---

# JSX là gì?

JSX cho phép bạn viết cú pháp giống HTML bên trong JavaScript.

Ví dụ:

```jsx
function App() {
  return <h1>Hello React</h1>;
}
```

React cho phép nhúng JavaScript trực tiếp vào JSX bằng dấu ngoặc nhọn `{}`.

---

# 1. Truyền chuỗi bằng dấu nháy `" "` hoặc `' '`

Khi truyền string tĩnh:

```jsx
<img
  src="avatar.png"
  alt="Ảnh đại diện"
/>
```

Ở đây:

```jsx
"avatar.png"
```

là string bình thường.

---

# 2. Dùng `{}` để đưa JavaScript vào JSX

## Ví dụ với biến

```jsx
export default function TodoList() {
  const name = 'Hedy Lamarr';

  return (
    <h1>{name}'s Todo List</h1>
  );
}
```

## Kết quả

```html
<h1>Hedy Lamarr's Todo List</h1>
```

---

# `{}` hoạt động như thế nào?

Dấu `{}` mở ra một “cửa sổ JavaScript” bên trong JSX.

Ví dụ:

```jsx
<h1>{name}</h1>
```

React hiểu rằng:

* `name` là biến JavaScript
* không phải text thông thường

---

# 3. Có thể dùng mọi JavaScript expression bên trong `{}`

## Ví dụ phép toán

```jsx
<p>{2 + 2}</p>
```

Kết quả:

```html
<p>4</p>
```

---

## Ví dụ gọi function

```jsx
const today = new Date();

function formatDate(date) {
  return date.getFullYear();
}

export default function App() {
  return (
    <h1>{formatDate(today)}</h1>
  );
}
```

---

# 4. Dùng `{}` trong attribute

## Ví dụ

```jsx
const imageUrl = 'avatar.png';

<img src={imageUrl} />
```

---

# So sánh cực kỳ quan trọng

## String bình thường

```jsx
<img src="imageUrl" />
```

=> React hiểu là chuỗi `"imageUrl"`

---

## Biến JavaScript

```jsx
<img src={imageUrl} />
```

=> React lấy giá trị của biến `imageUrl`

---

# 5. Nơi được phép dùng `{}`

React chỉ cho dùng `{}` ở 2 nơi:

---

## 1. Bên trong nội dung tag

✅ Đúng:

```jsx
<h1>{name}</h1>
```

❌ Sai:

```jsx
<{tag}>Hello</{tag}>
```

---

## 2. Sau dấu `=`

✅ Đúng:

```jsx
<img src={avatar} />
```

❌ Sai:

```jsx
<img src="{avatar}" />
```

Cái này sẽ thành string:

```txt
"{avatar}"
```

---

# 6. Double Curly Braces `{{ }}` là gì?

Đây là thứ làm nhiều người mới học React bị rối.

---

## Ví dụ

```jsx
<ul style={{
  backgroundColor: 'black',
  color: 'pink'
}}>
```

---

# Tại sao lại có `{{}}` ?

Vì:

## Cặp ngoài

```jsx
{}
```

=> mở JavaScript trong JSX

---

## Cặp trong

```jsx
{
  backgroundColor: 'black'
}
```

=> object JavaScript

---

# Nghĩa thật sự của đoạn code

```jsx
style={
  {
    backgroundColor: 'black',
    color: 'pink'
  }
}
```

---

# 7. Inline Style trong React

## HTML bình thường

```html
<ul style="background-color: black">
```

---

## React JSX

```jsx
<ul style={{ backgroundColor: 'black' }}>
```

---

# Lưu ý cực quan trọng

Trong React:

* CSS dùng camelCase
* không dùng kebab-case

---

## Ví dụ đúng

```jsx
backgroundColor
fontSize
marginTop
```

---

## Ví dụ sai

```jsx
background-color
font-size
```

---

# 8. Truyền object vào JSX

## Ví dụ

```jsx
const person = {
  name: 'Gregorio',
  age: 30
};

<h1>{person.name}</h1>
```

---

# 9. Dùng object cho style

```jsx
const person = {
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

<div style={person.theme}>
```

---

# 10. Lỗi rất hay gặp

## Sai

```jsx
<h1>{person}</h1>
```

Nếu `person` là object:

```jsx
const person = {
  name: 'David'
}
```

React sẽ báo lỗi:

```txt
Objects are not valid as a React child
```

---

# Cách đúng

```jsx
<h1>{person.name}</h1>
```

---

# 11. JSX cho phép dùng JavaScript expression

## Được phép

```jsx
{name}
{2 + 2}
{isLogin ? 'Yes' : 'No'}
{items.map(item => item.name)}
```

---

# Không được phép dùng statement

❌ Sai:

```jsx
{
  if (isLogin) {
    return <h1>Hello</h1>
  }
}
```

---

# Cách đúng

```jsx
{
  isLogin ? <h1>Hello</h1> : <h1>Bye</h1>
}
```

---

# 12. Comment trong JSX

Muốn comment trong JSX phải dùng:

```jsx
{/* Đây là comment */}
```

Vì JSX cần `{}` để hiểu đó là JavaScript.

---

# 13. Tổng kết

## JSX Attribute với dấu nháy

```jsx
src="avatar.png"
```

=> string

---

## JSX với `{}`

```jsx
src={avatar}
```

=> JavaScript variable

---

## `{{}}`

```jsx
style={{ color: 'red' }}
```

=> object JavaScript bên trong JSX

---

# Tư duy quan trọng cần nhớ

## JSX = HTML + JavaScript

Và:

```jsx
{}
```

chính là cây cầu nối JSX với JavaScript.

---

# Ví dụ tổng hợp

```jsx
export default function Profile() {
  const user = {
    name: 'David',
    age: 20,
    theme: {
      backgroundColor: 'black',
      color: 'white'
    }
  };

  return (
    <div style={user.theme}>
      <h1>{user.name}</h1>
      <p>Age: {user.age}</p>
      <p>{2 + 2}</p>
    </div>
  );
}
```

---

# Recap ngắn gọn

| Cú pháp                    | Ý nghĩa          |
| -------------------------- | ---------------- |
| `"text"`                   | String           |
| `{name}`                   | Biến JavaScript  |
| `{2 + 2}`                  | Expression       |
| `{formatDate()}`           | Gọi function     |
| `style={{ color: 'red' }}` | Object trong JSX |
| `{/* comment */}`          | Comment JSX      |

---

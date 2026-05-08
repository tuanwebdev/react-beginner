# Keeping Components Pure trong React

Nguồn chính:

* [React Docs - Keeping Components Pure](https://react.dev/learn/keeping-components-pure?utm_source=chatgpt.com)
* [React Docs - Components and Hooks Must Be Pure](https://react.dev/reference/rules/components-and-hooks-must-be-pure?utm_source=chatgpt.com)

---

# Pure Component là gì?

Trong React, component nên hoạt động giống **pure function**.

Một pure function có 2 đặc điểm:

1. **Same inputs → same output**
2. Không gây side effects ngoài phạm vi của nó

---

# Ví dụ pure function

```js
function double(number) {
  return number * 2;
}
```

---

# Kết quả luôn giống nhau

```js
double(2) // 4
double(2) // 4
double(2) // 4
```

Không bao giờ đổi.

---

# React Component cũng nên như vậy

```jsx
function Greeting({ name }) {
  return <h1>Hello {name}</h1>;
}
```

---

# Khi truyền cùng props

```jsx
<Greeting name="David" />
```

=> luôn render:

```html
<h1>Hello David</h1>
```

---

# Tư duy cực kỳ quan trọng

React muốn component giống:

```txt
(props, state, context) => UI
```

Nghĩa là:

* input giống nhau
* output JSX phải giống nhau

([React][1])

---

# 1. Components as Formulas

React xem component như công thức toán học.

---

# Ví dụ

genui{"math_block_widget_always_prefetch_v2":{"content":"y = 2x"}}

Nếu:

```txt
x = 2
```

=> luôn:

```txt
y = 4
```

---

# React cũng vậy

```jsx
function Recipe({ drinkers }) {
  return (
    <h1>{drinkers} cups of water</h1>
  );
}
```

---

# Kết quả

```jsx
<Recipe drinkers={2} />
```

=> luôn:

```html
<h1>2 cups of water</h1>
```

([React][1])

---

# 2. Side Effects là gì?

Side effect = hành động làm thay đổi thứ bên ngoài component.

Ví dụ:

* sửa biến global
* gọi API
* sửa DOM
* setTimeout
* localStorage
* mutate object ngoài scope

---

# Ví dụ side effect

```jsx
let guest = 0;

function Cup() {
  guest = guest + 1;

  return <h1>Guest #{guest}</h1>;
}
```

---

# Vì sao sai?

Component đang sửa biến ngoài component:

```js
guest
```

=> component không còn pure nữa.

([React][1])

---

# Kết quả nguy hiểm

```jsx
<Cup />
<Cup />
<Cup />
```

Có thể render:

```txt
Guest #1
Guest #2
Guest #3
```

hoặc:

```txt
Guest #2
Guest #4
Guest #6
```

trong StrictMode.

---

# 3. React StrictMode phát hiện component impure

React development mode có:

```jsx
<React.StrictMode>
```

---

# Nó sẽ gọi component 2 lần

để phát hiện:

* mutation
* side effects
* logic không predictable

([React][1])

---

# Vì vậy component impure dễ bị lỗi

---

# Ví dụ impure

```jsx
let count = 0;

function App() {
  count++;

  return <h1>{count}</h1>;
}
```

---

# StrictMode

Có thể render:

```txt
2
```

thay vì:

```txt
1
```

---

# 4. Cách đúng để giữ component pure

---

# Dùng props

```jsx
function Cup({ guest }) {
  return <h1>Guest #{guest}</h1>;
}
```

---

# Component cha

```jsx
function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

---

# Giờ component pure vì:

* không sửa biến ngoài
* output chỉ phụ thuộc props

([React][1])

---

# 5. Component không nên phụ thuộc render order

React có thể:

* render trước
* render lại
* pause rendering
* restart rendering

bất kỳ lúc nào.

---

# Vì vậy

❌ Không được giả định:

```txt
Component A render trước B
```

---

# React rendering phải độc lập

Mỗi component phải:

```txt
tự tính JSX của chính nó
```

([React][1])

---

# 6. Props, State, Context phải immutable

---

# Không được mutate props

❌ Sai:

```jsx
function Profile(props) {
  props.name = 'David';
}
```

---

# Không được mutate state trực tiếp

❌ Sai:

```jsx
user.name = 'David';
```

---

# Đúng

```jsx
setUser({
  ...user,
  name: 'David'
});
```

([React][2])

---

# 7. Local Mutation được phép

Đây là phần nhiều người bị nhầm.

---

# React cho phép mutate dữ liệu LOCAL

---

# Ví dụ đúng

```jsx
function TeaGathering() {
  const cups = [];

  for (let i = 1; i <= 3; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }

  return cups;
}
```

---

# Vì sao đúng?

Vì:

```js
const cups = [];
```

được tạo bên trong component.

Không ai bên ngoài biết.

---

# Đây gọi là

```txt
local mutation
```

([React][1])

---

# 8. Side Effects nên ở đâu?

---

# React khuyên:

## Ưu tiên Event Handlers

```jsx
<button onClick={handleClick}>
```

---

# Vì event handler KHÔNG chạy khi render

---

# Ví dụ

```jsx
function App() {
  function handleClick() {
    console.log('Clicked');
  }

  return (
    <button onClick={handleClick}>
      Click
    </button>
  );
}
```

---

# 9. `useEffect` cho side effects

Nếu không thể dùng event handler:

=> dùng `useEffect`.

---

# Ví dụ

```jsx
useEffect(() => {
  document.title = 'Hello';
}, []);
```

---

# Vì sao?

`useEffect`

* chạy SAU render
* không phá purity của render

([React][1])

---

# 10. Không được sửa DOM trong render

---

# Sai

```jsx
function Clock() {
  document.body.style.background = 'black';

  return <h1>Hello</h1>;
}
```

---

# Vì render phải chỉ return JSX

---

# Đúng

```jsx
useEffect(() => {
  document.body.style.background = 'black';
}, []);
```

([React][1])

---

# 11. Vì sao React quan tâm purity?

Purity giúp React:

---

# 1. Re-render an toàn

React có thể:

* pause
* retry
* restart

rendering.

---

# 2. Performance Optimization

React có thể:

* skip render
* cache render
* memoization

---

# 3. Concurrent Rendering

React 18+ hỗ trợ:

* concurrent rendering
* interrupt rendering

Purity làm điều đó an toàn.

([React][1])

---

# 12. `PureComponent` là gì?

Đây là class component optimization.

---

# Ví dụ

```jsx
import { PureComponent } from 'react';

class Greeting extends PureComponent {
  render() {
    return <h1>Hello</h1>;
  }
}
```

---

# `PureComponent` sẽ:

* shallow compare props/state
* skip unnecessary re-render

([React][3])

---

# Function component tương đương

```jsx
React.memo()
```

---

# Ví dụ

```jsx
const Greeting = React.memo(function Greeting() {
  return <h1>Hello</h1>;
});
```

([Reddit][4])

---

# 13. Function Component có phải pure không?

Câu trả lời:

```txt
NÊN pure
```

chứ không tự động pure.

---

# Bạn vẫn có thể viết impure component

```jsx
let count = 0;

function App() {
  count++;
}
```

---

# React docs muốn bạn:

```txt
viết component như pure function
```

([React][2])

---

# 14. Common Mistakes

---

# ❌ Mutate global variable

```jsx
count++;
```

---

# ❌ Mutate props

```jsx
props.user.name = 'David';
```

---

# ❌ Mutate state

```jsx
user.name = 'David';
```

---

# ❌ Side effects trong render

```jsx
localStorage.setItem()
fetch()
document.querySelector()
```

---

# ❌ Random values trong render

```jsx
Math.random()
Date.now()
```

---

# Vì:

same input ≠ same output

---

# 15. Ví dụ component PURE

```jsx
function Profile({ user }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.age}</p>
    </div>
  );
}
```

---

# Đặc điểm

✅ Không mutate gì
✅ Không side effect
✅ Same props → same JSX

---

# 16. Ví dụ component IMPURE

```jsx
let visits = 0;

function Profile({ user }) {
  visits++;

  return (
    <h1>
      {user.name} visited {visits}
    </h1>
  );
}
```

---

# Vì sao impure?

Render làm thay đổi:

```js
visits
```

---

# 17. Pure Rendering Logic

React muốn:

```txt
Render = chỉ tính toán UI
```

KHÔNG phải:

```txt
Render = thay đổi hệ thống
```

---

# Rendering nên giống

```txt
calculation
```

---

# Không nên giống

```txt
action
```

---

# 18. Recap ngắn gọn

| Khái niệm           | Ý nghĩa                                  |
| ------------------- | ---------------------------------------- |
| Pure component      | Same input → same output                 |
| Side effect         | Thay đổi bên ngoài component             |
| StrictMode          | Detect impure render                     |
| Props/state/context | Read-only                                |
| Local mutation      | Được phép                                |
| Event handler       | Nơi tốt cho side effects                 |
| useEffect           | Side effects sau render                  |
| React.memo          | Pure optimization cho function component |
| PureComponent       | Pure optimization cho class component    |

---

# Tư duy quan trọng nhất

## React muốn component giống:

UI = f(props, state, context)

Nghĩa là:

* UI chỉ là kết quả tính toán từ dữ liệu
* không được gây tác dụng phụ trong render

---

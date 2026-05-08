# Understanding Your UI as a Tree trong React

Nguồn chính:

* [React Docs - Understanding Your UI as a Tree](https://react.dev/learn/understanding-your-ui-as-a-tree?utm_source=chatgpt.com)
* [React Docs - Describing the UI](https://react.dev/learn/describing-the-ui?utm_source=chatgpt.com)

React mô hình hóa UI dưới dạng **tree (cây)** để quản lý:

* component
* render
* data flow
* performance
* state management

([React][1])

---

# Vì sao React dùng Tree?

UI tự nhiên có cấu trúc phân cấp.

Ví dụ:

```txt id="tree1"
App
 ├── Header
 │    ├── Logo
 │    └── Navbar
 │
 ├── Main
 │    ├── Sidebar
 │    └── Content
 │
 └── Footer
```

---

# Đây chính là Component Tree

Mỗi component:

* có parent
* có child
* tạo thành cấu trúc cây

---

# Tư duy cực kỳ quan trọng

React không nhìn app như:

```txt id="tree2"
1 file lớn
```

Mà nhìn như:

```txt id="tree3"
nhiều component lồng nhau
```

---

# 1. Tree là gì?

Tree là cấu trúc dữ liệu gồm:

* root node
* parent node
* child node
* leaf node

---

# Ví dụ

```txt id="tree4"
        App
       /   \
   Header  Main
            |
         Sidebar
```

---

# Giải thích

| Node      | Ý nghĩa        |
| --------- | -------------- |
| `App`     | Root           |
| `Header`  | Child của App  |
| `Main`    | Child của App  |
| `Sidebar` | Child của Main |
| `Sidebar` | Leaf node      |

---

# 2. React Render Tree

Render Tree = cây thể hiện component nào render component nào.

---

# Ví dụ

```jsx id="tree5"
function App() {
  return (
    <>
      <Header />
      <Main />
    </>
  );
}
```

---

# `Main`

```jsx id="tree6"
function Main() {
  return (
    <>
      <Sidebar />
      <Content />
    </>
  );
}
```

---

# Render Tree

```txt id="tree7"
App
 ├── Header
 └── Main
      ├── Sidebar
      └── Content
```

([React][1])

---

# React tạo Render Tree như thế nào?

React sẽ:

```txt id="tree8"
Render App
 ↓
thấy <Header />
 ↓
render Header
 ↓
thấy <Main />
 ↓
render Main
 ↓
...
```

---

# Flow thực tế

```txt id="tree9"
Component
 ↓
React Render Tree
 ↓
DOM Tree
 ↓
Browser render UI
```

([React][1])

---

# 3. Root Component

Đây là component đầu tiên React render.

---

# Ví dụ

```jsx id="tree10"
function App() {
  return <Dashboard />;
}
```

---

# Tree

```txt id="tree11"
App
 └── Dashboard
```

---

# `App`

là root component.

---

# 4. Parent và Child Component

---

# Ví dụ

```jsx id="tree12"
function App() {
  return <Profile />;
}
```

---

# `App`

là parent của:

```jsx id="tree13"
<Profile />
```

---

# `Profile`

là child của `App`.

---

# Tree

```txt id="tree14"
App
 └── Profile
```

---

# 5. Leaf Components

Leaf = component không có child.

---

# Ví dụ

```txt id="tree15"
App
 ├── Header
 └── Footer
```

Nếu:

* `Header` không render component khác
* `Footer` không render component khác

=> chúng là leaf components.

([React][1])

---

# Vì sao leaf component quan trọng?

Leaf component thường:

✅ nhỏ
✅ re-render nhiều
✅ dễ optimize

---

# 6. Top-level Components

Top-level components gần root.

---

# Ví dụ

```txt id="tree16"
App
 ├── Dashboard
 ├── Sidebar
 └── Header
```

---

# Những component này ảnh hưởng lớn đến performance

Vì khi chúng render lại:

```txt id="tree17"
toàn bộ subtree bên dưới cũng có thể render lại
```

([React][1])

---

# 7. Conditional Rendering làm Tree thay đổi

---

# Ví dụ

```jsx id="tree18"
function App({ isLogin }) {
  return (
    <>
      {isLogin
        ? <Dashboard />
        : <Login />
      }
    </>
  );
}
```

---

# Khi `isLogin = true`

```txt id="tree19"
App
 └── Dashboard
```

---

# Khi `isLogin = false`

```txt id="tree20"
App
 └── Login
```

---

# Tree có thể thay đổi theo mỗi render

([React][1])

---

# 8. Render Tree ≠ DOM Tree

Đây là phần QUAN TRỌNG.

---

# Render Tree

```txt id="tree21"
React Components
```

---

# DOM Tree

```txt id="tree22"
HTML Elements
```

---

# Ví dụ

```jsx id="tree23"
function App() {
  return (
    <div>
      <h1>Hello</h1>
    </div>
  );
}
```

---

# React Render Tree

```txt id="tree24"
App
```

---

# DOM Tree

```html id="tree25"
div
 └── h1
```

---

# Vì sao?

Render tree chỉ chứa:

```txt id="tree26"
React components
```

KHÔNG chứa HTML tags.

([React][1])

---

# 9. React là Platform Agnostic

React không chỉ render web.

React có thể render:

* web
* mobile
* desktop

---

# Web

```txt id="tree27"
DOM
```

---

# React Native

```txt id="tree28"
UIView
```

---

# Vì vậy React tree không phụ thuộc HTML.

([React][1])

---

# 10. Module Dependency Tree

Ngoài render tree, React còn có:

```txt id="tree29"
Module Dependency Tree
```

---

# Đây là cây import/export.

---

# Ví dụ

```jsx id="tree30"
import Header from './Header';
import Main from './Main';
```

---

# Dependency Tree

```txt id="tree31"
App.js
 ├── Header.js
 └── Main.js
```

---

# Nếu `Main.js`

```jsx id="tree32"
import Sidebar from './Sidebar';
```

---

# Tree

```txt id="tree33"
App.js
 ├── Header.js
 └── Main.js
       └── Sidebar.js
```

([React][1])

---

# 11. Render Tree vs Dependency Tree

| Render Tree       | Dependency Tree     |
| ----------------- | ------------------- |
| Quan hệ render    | Quan hệ import      |
| Node là component | Node là module/file |
| Runtime           | Build time          |
| Phụ thuộc JSX     | Phụ thuộc import    |

---

# Ví dụ cực kỳ quan trọng

---

# App.js

```jsx id="tree34"
import Copyright from './Copyright';

function App() {
  return (
    <Layout>
      <Copyright />
    </Layout>
  );
}
```

---

# Dependency Tree

```txt id="tree35"
App.js
 └── Copyright.js
```

---

# Nhưng Render Tree

```txt id="tree36"
App
 └── Layout
       └── Copyright
```

---

# Vì:

`Layout`

render:

```jsx id="tree37"
{children}
```

---

# Đây là khác biệt rất lớn

([React][1])

---

# 12. Vì sao Dependency Tree quan trọng?

Build tools như:

* Vite
* Webpack
* Parcel

dùng dependency tree để:

✅ bundle code
✅ tree shaking
✅ code splitting
✅ lazy loading

([React][1])

---

# 13. Tree Shaking

Bundler sẽ loại bỏ code không dùng.

---

# Ví dụ

```js id="tree38"
import { add } from './math';
```

Nếu không dùng:

```js id="tree39"
subtract
multiply
```

=> bundler có thể remove.

---

# Đây gọi là

```txt id="tree40"
Tree Shaking
```

---

# 14. Component Tree giúp Debug

Tree giúp bạn hiểu:

✅ data flow
✅ props flow
✅ state position
✅ re-render
✅ performance bottleneck

---

# Ví dụ

```txt id="tree41"
App
 └── Dashboard
      └── UserList
           └── UserCard
```

Nếu `App` re-render:

```txt id="tree42"
Dashboard
UserList
UserCard
```

cũng có thể re-render.

---

# Đây là lý do optimize tree quan trọng.

---

# 15. React DevTools hiển thị Tree

Extension:

[React Developer Tools](https://react.dev/learn/react-developer-tools?utm_source=chatgpt.com)

cho phép xem:

* component tree
* props
* state
* re-render

---

# Ví dụ

```txt id="tree43"
<App>
  <Header>
  <Sidebar>
  <Dashboard>
```

---

# Đây là tool cực kỳ quan trọng cho React dev.

---

# 16. State gắn với vị trí trong Tree

Đây là concept QUAN TRỌNG.

React giữ state theo:

```txt id="tree44"
position in the tree
```

---

# Ví dụ

```jsx id="tree45"
{
  show
    ? <Counter />
    : <Counter />
}
```

React có thể giữ state.

---

# Nhưng nếu đổi position:

```jsx id="tree46"
{
  show
    ? <Counter />
    : <Other />
}
```

=> state reset.

([Reddit][2])

---

# 17. Thinking in React

Community thường khuyên:

```txt id="tree47"
Think your app as a tree
```

([Reddit][3])

---

# Ví dụ

```txt id="tree48"
App
 ├── Layout
 │    ├── Navbar
 │    └── Sidebar
 │
 └── Pages
      ├── Home
      └── About
```

---

# Sau đó mới:

* đặt state
* truyền props
* chia component

---

# 18. Performance và Tree

Khi tree quá lớn:

```txt id="tree49"
1000+
components
```

React phải:

* diff tree
* reconcile
* update DOM

---

# Vì vậy React optimize bằng:

* Fiber
* memo
* virtualization
* concurrent rendering

([Reddit][4])

---

# 19. React Fiber

React internally dùng:

```txt id="tree50"
Fiber Tree
```

---

# Fiber giúp:

✅ pause rendering
✅ prioritize updates
✅ concurrent rendering
✅ interruption

---

# Đây là core architecture của React 16+.

---

# 20. Ví dụ tổng hợp

```jsx id="tree51"
function App() {
  return (
    <Layout>
      <Header />
      <Main />
    </Layout>
  );
}

function Main() {
  return (
    <>
      <Sidebar />
      <Content />
    </>
  );
}
```

---

# Render Tree

```txt id="tree52"
App
 └── Layout
      ├── Header
      └── Main
           ├── Sidebar
           └── Content
```

---

# Dependency Tree

```txt id="tree53"
App.js
 ├── Layout.js
 ├── Header.js
 ├── Main.js
 ├── Sidebar.js
 └── Content.js
```

---

# Recap ngắn gọn

| Khái niệm             | Ý nghĩa                         |
| --------------------- | ------------------------------- |
| Tree                  | Quan hệ phân cấp                |
| Render Tree           | Component render component      |
| Dependency Tree       | File import file                |
| Root Component        | Component đầu tiên              |
| Parent/Child          | Quan hệ component               |
| Leaf Component        | Không có child                  |
| Conditional Rendering | Tree thay đổi                   |
| React DevTools        | Xem component tree              |
| State Position        | State gắn với vị trí tree       |
| Fiber Tree            | Internal architecture của React |

---

# Tư duy quan trọng nhất

React nhìn app như:

```txt id="tree54"
UI Tree
```

KHÔNG phải:

```txt id="tree55"
1 file lớn
```

---

# Mental Model cực kỳ quan trọng

## React App = Tree of Components

```txt id="tree56"
App
 ├── Header
 ├── Sidebar
 ├── Main
 │    ├── Card
 │    └── List
 └── Footer
```

Hiểu được tree này sẽ giúp bạn:

✅ chia component tốt hơn
✅ quản lý state dễ hơn
✅ optimize performance tốt hơn
✅ debug dễ hơn

---


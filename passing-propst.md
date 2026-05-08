# Passing Props to a Component trong React

Nguồn chính: [React Docs - Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component?utm_source=chatgpt.com) ([it.react.dev][1])

---

# Props là gì?

Props (properties) là cách các component React giao tiếp với nhau.

* Component cha → truyền dữ liệu xuống component con
* Component con → nhận dữ liệu qua `props`

Props giống tham số của function.

---

# Ví dụ đơn giản nhất

```jsx
function Welcome(props) {
  return <h1>Hello {props.name}</h1>;
}

export default function App() {
  return <Welcome name="David" />;
}
```

---

# Kết quả

```html
<h1>Hello David</h1>
```

---

# Tư duy cực kỳ quan trọng

```jsx
<Welcome name="David" />
```

giống như:

```js
Welcome({ name: "David" })
```

---

# 1. Props quen thuộc trong JSX

Các attribute HTML thật ra cũng là props.

Ví dụ:

```jsx
<img
  src="avatar.png"
  alt="Avatar"
  width={100}
  height={100}
/>
```

Ở đây:

* `src`
* `alt`
* `width`
* `height`

đều là props.

([it.react.dev][1])

---

# 2. Truyền props vào component

## Component cha

```jsx
export default function Profile() {
  return (
    <Avatar
      person={{
        name: 'Lin Lanying',
        imageId: '1bX5QH6'
      }}
      size={100}
    />
  );
}
```

---

# Component con nhận props

```jsx
function Avatar(props) {
  return (
    <h1>{props.person.name}</h1>
  );
}
```

---

# Props nhận được là object

```js
props = {
  person: {
    name: 'Lin Lanying',
    imageId: '1bX5QH6'
  },
  size: 100
}
```

---

# 3. Destructuring Props

Cách viết phổ biến hơn:

```jsx
function Avatar({ person, size }) {
  return (
    <h1>{person.name}</h1>
  );
}
```

---

# Nó tương đương với

```jsx
function Avatar(props) {
  const person = props.person;
  const size = props.size;

  return (
    <h1>{person.name}</h1>
  );
}
```

([it.react.dev][1])

---

# Lưu ý cực kỳ quan trọng

## Đúng

```jsx
function Avatar({ person, size }) {

}
```

---

## Sai

```jsx
function Avatar(person, size) {

}
```

React component chỉ nhận **1 tham số duy nhất** là object props.

---

# 4. Truyền nhiều loại dữ liệu khác nhau

---

# String

```jsx
<Button text="Save" />
```

---

# Number

```jsx
<Button size={20} />
```

---

# Boolean

```jsx
<Button disabled={true} />
```

Có thể viết ngắn:

```jsx
<Button disabled />
```

---

# Array

```jsx
<List items={['React', 'Vue', 'Angular']} />
```

---

# Object

```jsx
<User person={{ name: 'David', age: 20 }} />
```

---

# Function

```jsx
<Button onClick={handleClick} />
```

---

# JSX

```jsx
<Card content={<h1>Hello</h1>} />
```

---

# 5. Props là Read-only

Props KHÔNG được sửa trực tiếp.

---

# Sai

```jsx
function Avatar(props) {
  props.name = "David";
}
```

---

# Vì sao?

Props là dữ liệu do component cha truyền xuống.

Component con chỉ được đọc.

([it.react.dev][1])

---

# 6. Default Props

Bạn có thể đặt giá trị mặc định.

---

# Ví dụ

```jsx
function Avatar({ size = 100 }) {
  return <h1>{size}</h1>;
}
```

---

# Khi không truyền props

```jsx
<Avatar />
```

=> `size = 100`

---

# Khi truyền props

```jsx
<Avatar size={200} />
```

=> `size = 200`

---

# Lưu ý quan trọng

Default chỉ hoạt động khi:

```jsx
size === undefined
```

---

# Ví dụ

## Dùng default

```jsx
<Avatar size={undefined} />
```

---

## Không dùng default

```jsx
<Avatar size={null} />
<Avatar size={0} />
```

([it.react.dev][1])

---

# 7. JSX Spread Syntax

---

# Bình thường

```jsx
<Avatar
  person={person}
  size={size}
  isAdmin={isAdmin}
/>
```

---

# Dùng spread

```jsx
const props = {
  person,
  size,
  isAdmin
};

<Avatar {...props} />
```

---

# Nó tương đương với

```jsx
<Avatar
  person={props.person}
  size={props.size}
  isAdmin={props.isAdmin}
/>
```

([it.react.dev][1])

---

# Khi nào nên dùng spread?

## Nên dùng

Khi forwarding props:

```jsx
function Card(props) {
  return <Profile {...props} />
}
```

---

## Không nên lạm dụng

Vì khó biết component nhận prop gì.

---

# 8. Props Children

Một component có thể nhận JSX bên trong tag.

---

# Ví dụ

```jsx
<Card>
  <h1>Hello</h1>
  <p>React</p>
</Card>
```

---

# React sẽ tạo

```js
props = {
  children: (
    <>
      <h1>Hello</h1>
      <p>React</p>
    </>
  )
}
```

---

# Component nhận children

```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}
```

---

# Đây là pattern cực kỳ quan trọng trong React

Ví dụ:

* Modal
* Layout
* Sidebar
* Wrapper component

đều dùng `children`.

([it.react.dev][1])

---

# 9. Props thay đổi theo thời gian

Props không cố định.

Component cha render lại → props mới được truyền xuống.

---

# Ví dụ

```jsx
function Clock({ time, color }) {
  return (
    <h1 style={{ color }}>
      {time}
    </h1>
  );
}
```

---

# Khi state cha đổi

* `time` đổi
* `color` đổi

=> component con render lại với props mới.

([it.react.dev][1])

---

# 10. Props vs State

| Props                        | State                      |
| ---------------------------- | -------------------------- |
| Truyền từ cha                | Dữ liệu nội bộ             |
| Read-only                    | Có thể thay đổi            |
| Component con không sửa được | Component tự cập nhật được |
| Dùng để giao tiếp            | Dùng để lưu trạng thái     |

---

# 11. Special Props

Có 2 props đặc biệt:

```jsx
key
ref
```

React dùng nội bộ nên component KHÔNG đọc được trực tiếp.

---

# Ví dụ

```jsx
<Item key={id} />
```

Trong component:

```jsx
props.key
```

=> `undefined`

([React][2])

---

# Nếu muốn dùng key bên trong component

Phải truyền thêm prop khác:

```jsx
<Item key={id} itemId={id} />
```

---

# 12. Props Drilling

Khi props bị truyền qua nhiều tầng component.

---

# Ví dụ

```jsx
<App user={user} />
```

↓

```jsx
<Layout user={user} />
```

↓

```jsx
<Sidebar user={user} />
```

↓

```jsx
<Profile user={user} />
```

---

# Đây gọi là Props Drilling

Khi quá sâu → thường dùng:

* Context API
* Redux
* Zustand

---

# 13. Truyền function qua props

Đây là cách component con giao tiếp ngược lên cha.

---

# Ví dụ

```jsx
function Parent() {
  function handleClick() {
    console.log('Clicked');
  }

  return <Button onClick={handleClick} />;
}
```

---

# Component con

```jsx
function Button({ onClick }) {
  return (
    <button onClick={onClick}>
      Click me
    </button>
  );
}
```

---

# Tư duy quan trọng

* Dữ liệu: cha → con
* Event/function: con → cha

---

# 14. Props thực chất là function arguments

Nhiều dev React coi:

```jsx
<Component />
```

là:

```js
Component(props)
```

([Reddit][3])

---

# Ví dụ tổng hợp

```jsx
function UserCard({
  name,
  age,
  isOnline = false,
  children
}) {
  return (
    <div>
      <h1>{name}</h1>
      <p>{age}</p>

      {isOnline && <p>Online</p>}

      {children}
    </div>
  );
}

export default function App() {
  return (
    <UserCard
      name="David"
      age={20}
      isOnline={true}
    >
      <button>Add Friend</button>
    </UserCard>
  );
}
```

---

# Recap ngắn gọn

| Khái niệm           | Ý nghĩa                         |
| ------------------- | ------------------------------- |
| Props               | Dữ liệu truyền từ cha xuống con |
| `props.name`        | Đọc props                       |
| `{ name }`          | Destructuring                   |
| `size = 100`        | Default prop                    |
| `{...props}`        | Spread props                    |
| `children`          | JSX bên trong component         |
| Props are read-only | Không được sửa props            |
| key/ref             | Special props                   |



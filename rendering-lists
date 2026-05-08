# Rendering Lists trong React

Nguồn chính: [React Docs - Rendering Lists](https://react.dev/learn/rendering-lists?utm_source=chatgpt.com) ([React][1])

---

# Rendering Lists là gì?

Trong React, bạn thường cần hiển thị nhiều component giống nhau từ một mảng dữ liệu.

Ví dụ:

* danh sách sản phẩm
* comment
* bài viết
* user
* notification

React thường dùng:

* `map()`
* `filter()`

để render list. ([React][1])

---

# Tư duy cực kỳ quan trọng

Trong React:

```jsx id="rl1"
const listItems = items.map(item =>
  <li>{item}</li>
);
```

nghĩa là:

* duyệt từng phần tử trong array
* biến mỗi phần tử thành JSX
* React render toàn bộ JSX đó

---

# 1. Render list cơ bản với `map()`

---

# Dữ liệu

```jsx id="rl2"
const people = [
  'David',
  'John',
  'Anna'
];
```

---

# Render

```jsx id="rl3"
export default function List() {
  const listItems = people.map(person =>
    <li>{person}</li>
  );

  return <ul>{listItems}</ul>;
}
```

---

# Kết quả

```html id="rl4"
<ul>
  <li>David</li>
  <li>John</li>
  <li>Anna</li>
</ul>
```

([React][1])

---

# `map()` hoạt động như thế nào?

---

# JavaScript bình thường

```js id="rl5"
const numbers = [1, 2, 3];

const doubled = numbers.map(n => n * 2);

console.log(doubled);
```

---

# Kết quả

```js id="rl6"
[2, 4, 6]
```

---

# Trong React

```jsx id="rl7"
people.map(person =>
  <li>{person}</li>
)
```

=> tạo array JSX.

---

# 2. Render object list

Thông thường list sẽ là array object.

---

# Ví dụ

```jsx id="rl8"
const people = [
  {
    id: 1,
    name: 'David',
    profession: 'Developer'
  },
  {
    id: 2,
    name: 'Anna',
    profession: 'Designer'
  }
];
```

---

# Render

```jsx id="rl9"
function List() {
  const listItems = people.map(person =>
    <li key={person.id}>
      <h1>{person.name}</h1>
      <p>{person.profession}</p>
    </li>
  );

  return <ul>{listItems}</ul>;
}
```

---

# Kết quả

```html id="rl10"
<li>
  <h1>David</h1>
  <p>Developer</p>
</li>
```

---

# 3. `key` là gì?

Đây là phần QUAN TRỌNG NHẤT của rendering lists.

---

# React yêu cầu mỗi item có `key`

---

# Sai

```jsx id="rl11"
people.map(person =>
  <li>{person.name}</li>
)
```

React warning:

```txt id="rl12"
Each child in a list should have a unique "key" prop.
```

([React][1])

---

# Đúng

```jsx id="rl13"
people.map(person =>
  <li key={person.id}>
    {person.name}
  </li>
)
```

---

# Vì sao React cần `key`?

`key` giúp React:

* nhận diện item
* tối ưu re-render
* biết item nào:

  * thêm
  * xóa
  * đổi vị trí

([React][1])

---

# Ví dụ dễ hiểu

---

# Không có key

```txt id="rl14"
A
B
C
```

Xóa `B`

```txt id="rl15"
A
C
```

React có thể nhầm:

* B thành C

---

# Có key

```txt id="rl16"
id=1
id=2
id=3
```

React biết chính xác:

* id=2 bị xóa

---

# 4. Rules của `key`

---

# Key phải unique giữa siblings

✅ Đúng

```jsx id="rl17"
<li key={person.id}>
```

---

# Không nên dùng index

❌ Tránh:

```jsx id="rl18"
items.map((item, index) =>
  <li key={index}>
```

---

# Vì sao?

Nếu list:

* sort
* insert
* delete

React sẽ render sai item.

([React][1])

---

# Không dùng random key

❌ Sai:

```jsx id="rl19"
<li key={Math.random()}>
```

---

# Vì:

Mỗi render key đều đổi.

React sẽ:

* destroy component cũ
* tạo component mới

=> mất performance + mất state/input.

([React][1])

---

# Best Practice cho key

| Trường hợp    | Key nên dùng          |
| ------------- | --------------------- |
| Database data | `id`                  |
| Local data    | UUID                  |
| Static list   | index (tạm chấp nhận) |

---

# 5. Filtering Lists với `filter()`

---

# Dữ liệu

```jsx id="rl20"
const people = [
  {
    id: 1,
    name: 'David',
    profession: 'Developer'
  },
  {
    id: 2,
    name: 'Anna',
    profession: 'Designer'
  }
];
```

---

# Chỉ lấy developer

```jsx id="rl21"
const developers = people.filter(
  person => person.profession === 'Developer'
);
```

---

# Render

```jsx id="rl22"
const listItems = developers.map(person =>
  <li key={person.id}>
    {person.name}
  </li>
);
```

([React][1])

---

# 6. Kết hợp `filter()` + `map()`

Đây là pattern React dùng rất nhiều.

---

# Ví dụ

```jsx id="rl23"
const listItems = people
  .filter(person => person.profession === 'Developer')
  .map(person =>
    <li key={person.id}>
      {person.name}
    </li>
  );
```

---

# Flow hoạt động

```txt id="rl24"
Array
 ↓
filter()
 ↓
map()
 ↓
JSX
 ↓
Render
```

---

# 7. Arrow Function Pitfall

---

# Implicit return

✅ Đúng:

```jsx id="rl25"
people.map(person =>
  <li>{person.name}</li>
)
```

---

# Nếu dùng `{}` phải có `return`

✅ Đúng:

```jsx id="rl26"
people.map(person => {
  return <li>{person.name}</li>;
})
```

---

# Sai

```jsx id="rl27"
people.map(person => {
  <li>{person.name}</li>
})
```

=> render NOTHING.

([React][1])

---

# Vì sao?

---

# Arrow function có 2 kiểu

## Expression body

```js id="rl28"
x => x * 2
```

=> auto return

---

## Block body

```js id="rl29"
x => {
  return x * 2
}
```

=> phải tự return

---

# 8. Rendering nhiều DOM nodes

---

# Sai

```jsx id="rl30"
people.map(person =>
  <h1>{person.name}</h1>
  <p>{person.bio}</p>
)
```

JSX chỉ return 1 root element.

---

# Cách đúng với Fragment

```jsx id="rl31"
import { Fragment } from 'react';

people.map(person =>
  <Fragment key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </Fragment>
)
```

---

# Hoặc

```jsx id="rl32"
people.map(person =>
  <div key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </div>
)
```

([React][1])

---

# Lưu ý về Fragment shorthand

---

# Shorthand

```jsx id="rl33"
<>
  <h1>Hello</h1>
</>
```

---

# Nhưng shorthand KHÔNG nhận key

❌ Sai:

```jsx id="rl34"
<>
  ...
</>
```

trong `.map()`

---

# Phải dùng

```jsx id="rl35"
<Fragment key={id}>
```

([React][1])

---

# 9. Tách component cho list item

Pattern rất phổ biến.

---

# Component

```jsx id="rl36"
function Person({ person }) {
  return (
    <li>
      {person.name}
    </li>
  );
}
```

---

# Render list

```jsx id="rl37"
function List({ people }) {
  return (
    <ul>
      {people.map(person =>
        <Person
          key={person.id}
          person={person}
        />
      )}
    </ul>
  );
}
```

---

# Lưu ý cực kỳ quan trọng

`key` đặt ở nơi `.map()` xảy ra.

---

# Đúng

```jsx id="rl38"
people.map(person =>
  <Person key={person.id} />
)
```

---

# Sai

```jsx id="rl39"
function Person({ person }) {
  return (
    <li key={person.id}>
```

([Reddit][2])

---

# Vì sao?

`key` dành cho React quản lý LIST.

Không phải component con.

---

# 10. Nested Lists

---

# Ví dụ

```jsx id="rl40"
const categories = [
  {
    id: 1,
    name: 'Frontend',
    skills: ['React', 'Vue']
  }
];
```

---

# Render nested list

```jsx id="rl41"
categories.map(category =>
  <div key={category.id}>
    <h1>{category.name}</h1>

    <ul>
      {category.skills.map(skill =>
        <li key={skill}>{skill}</li>
      )}
    </ul>
  </div>
)
```

---

# 11. Conditional Rendering trong List

---

# Ví dụ

```jsx id="rl42"
people.map(person =>
  person.isOnline
    ? <li key={person.id}>{person.name}</li>
    : null
)
```

---

# Hoặc

```jsx id="rl43"
people
  .filter(person => person.isOnline)
  .map(person =>
    <li key={person.id}>
      {person.name}
    </li>
  )
```

---

# Best Practice

✅ Ưu tiên `filter()` trước `map()`

Dễ đọc hơn.

---

# 12. Performance với List lớn

Khi render:

```txt id="rl44"
1000+
10000+
```

items:

* app có thể lag
* render chậm

---

# Community thường dùng

* react-window
* react-virtualized
* virtualization

để chỉ render item đang visible. ([Reddit][3])

---

# 13. React Render Flow với List

React sẽ:

```txt id="rl45"
map()
 ↓
create JSX
 ↓
compare key
 ↓
update DOM
```

Nếu key ổn định:

✅ React update cực nhanh.

([React][4])

---

# 14. Common Mistakes

---

# ❌ Quên key

```jsx id="rl46"
items.map(item =>
  <li>{item}</li>
)
```

---

# ❌ Dùng random key

```jsx id="rl47"
key={Math.random()}
```

---

# ❌ Dùng index cho dynamic list

```jsx id="rl48"
key={index}
```

---

# ❌ Quên return

```jsx id="rl49"
items.map(item => {
  <li>{item}</li>
})
```

---

# ❌ Render object trực tiếp

```jsx id="rl50"
<li>{person}</li>
```

---

# Đúng

```jsx id="rl51"
<li>{person.name}</li>
```

---

# 15. Ví dụ tổng hợp

```jsx id="rl52"
const people = [
  {
    id: 1,
    name: 'David',
    profession: 'Developer'
  },
  {
    id: 2,
    name: 'Anna',
    profession: 'Designer'
  },
  {
    id: 3,
    name: 'John',
    profession: 'Developer'
  }
];

export default function List() {
  const developers = people.filter(
    person => person.profession === 'Developer'
  );

  return (
    <ul>
      {developers.map(person =>
        <li key={person.id}>
          <h1>{person.name}</h1>
          <p>{person.profession}</p>
        </li>
      )}
    </ul>
  );
}
```

---

# Flow của ví dụ trên

```txt id="rl53"
people
 ↓
filter()
 ↓
developers only
 ↓
map()
 ↓
JSX array
 ↓
render
```

---

# Recap ngắn gọn

| Khái niệm        | Ý nghĩa               |
| ---------------- | --------------------- |
| `map()`          | Transform array → JSX |
| `filter()`       | Lọc dữ liệu           |
| `key`            | Định danh item        |
| `Fragment`       | Group nhiều node      |
| `key={id}`       | Best practice         |
| `key={index}`    | Tránh dùng            |
| `filter().map()` | Pattern phổ biến      |

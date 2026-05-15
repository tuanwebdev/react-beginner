# Scaling Up with Reducer and Context — React

Nguồn: [React Docs - Scaling Up with Reducer and Context](https://react.dev/learn/scaling-up-with-reducer-and-context?utm_source=chatgpt.com)

---

# 1. Mục tiêu của bài học

React hướng dẫn cách:

* Kết hợp `useReducer` + `Context`
* Tránh truyền props quá sâu (`prop drilling`)
* Tổ chức state lớn rõ ràng hơn
* Tách logic state khỏi UI component

---

# 2. Vấn đề cần giải quyết

Khi app nhỏ:

```jsx
<App>
  <TaskList tasks={tasks} />
</App>
```

→ truyền props bình thường là đủ.

---

Nhưng khi app lớn:

```jsx
<App>
  <Layout>
    <Sidebar>
      <Panel>
        <TaskList />
      </Panel>
    </Sidebar>
  </Layout>
</App>
```

Nếu `TaskList` cần state:

* phải truyền props qua rất nhiều component trung gian
* gọi là:

# Prop Drilling

Ví dụ:

```jsx
<App tasks={tasks}>
  <Layout tasks={tasks}>
    <Sidebar tasks={tasks}>
      <Panel tasks={tasks}>
        <TaskList tasks={tasks} />
      </Panel>
    </Sidebar>
  </Layout>
</App>
```

Rất khó maintain.

---

# 3. Ý tưởng chính của React

React đề xuất:

| Hook         | Vai trò                         |
| ------------ | ------------------------------- |
| `useReducer` | quản lý logic state             |
| `Context`    | truyền state toàn cây component |

Kết hợp lại:

```txt
Reducer = quản lý state
Context = phân phối state
```

---

# 4. useReducer dùng để làm gì?

`useReducer` giúp:

* gom logic update state vào 1 chỗ
* dễ quản lý state phức tạp
* thay nhiều `useState`

---

## Cú pháp

```jsx
const [state, dispatch] = useReducer(reducer, initialState)
```

---

## Reducer là gì?

Reducer là function nhận:

```jsx
(state, action)
```

và trả về:

```jsx
newState
```

---

## Ví dụ

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added':
      return [...tasks, action.task]

    case 'deleted':
      return tasks.filter(t => t.id !== action.id)

    default:
      throw Error('Unknown action')
  }
}
```

---

# 5. dispatch là gì?

`dispatch()` dùng để gửi action tới reducer.

Ví dụ:

```jsx
dispatch({
  type: 'added',
  task: newTask
})
```

Luồng hoạt động:

```txt
dispatch(action)
    ↓
reducer(state, action)
    ↓
newState
    ↓
React re-render
```

---

# 6. Context dùng để làm gì?

Context giúp:

* component con lấy data trực tiếp
* không cần truyền props qua nhiều tầng

---

## Không dùng Context

```txt
App
 ↓
Layout
 ↓
Sidebar
 ↓
TaskList
```

props phải đi xuyên qua tất cả.

---

## Dùng Context

```txt
Context Provider
      ↓
mọi component bên dưới đều truy cập được
```

---

# 7. React khuyên tạo 2 Context riêng

React docs tạo:

```jsx
export const TasksContext = createContext(null)
export const TasksDispatchContext = createContext(null)
```

---

## Vì sao tách riêng?

| Context              | Chứa              |
| -------------------- | ----------------- |
| TasksContext         | state             |
| TasksDispatchContext | dispatch function |

---

## Lợi ích

Component chỉ cần `dispatch`

→ không re-render khi state đổi. ([Reddit][1])

---

# 8. Các bước kết hợp Reducer + Context

---

# Bước 1 — Tạo Context

```jsx
import { createContext } from 'react'

export const TasksContext = createContext(null)
export const TasksDispatchContext = createContext(null)
```

---

# Bước 2 — Đưa state và dispatch vào Context

```jsx
<TasksContext value={tasks}>
  <TasksDispatchContext value={dispatch}>
    <App />
  </TasksDispatchContext>
</TasksContext>
```

---

# Bước 3 — Component con dùng useContext

## Đọc state

```jsx
const tasks = useContext(TasksContext)
```

---

## Dispatch action

```jsx
const dispatch = useContext(TasksDispatchContext)

dispatch({
  type: 'deleted',
  id: task.id
})
```

---

# 9. Luồng dữ liệu hoàn chỉnh

```txt
Component
   ↓ dispatch(action)

Reducer xử lý
   ↓

State mới
   ↓

Context cập nhật
   ↓

Component re-render
```

---

# 10. Tư duy cực kỳ quan trọng

## useReducer không thay Context

Reducer:

* chỉ quản lý logic state

Context:

* chỉ truyền state

Chúng giải quyết 2 vấn đề khác nhau.

---

# 11. Provider Pattern

React khuyên tạo component Provider riêng.

---

## Ví dụ

```jsx
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks)

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  )
}
```

---

## App trở nên sạch hơn

```jsx
<TasksProvider>
  <TaskApp />
</TasksProvider>
```

---

# 12. Custom Hook

React docs còn tạo:

```jsx
export function useTasks() {
  return useContext(TasksContext)
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext)
}
```

---

## Lợi ích

Component dễ đọc hơn:

```jsx
const tasks = useTasks()
const dispatch = useTasksDispatch()
```

thay vì:

```jsx
const tasks = useContext(TasksContext)
```

---

# 13. Kiến trúc thư mục nên nhớ

```txt
src/
 ├── context/
 │    └── TasksContext.js
 │
 ├── components/
 │    ├── AddTask.jsx
 │    └── TaskList.jsx
 │
 └── App.jsx
```

---

# 14. Khi nào nên dùng useReducer?

Nên dùng khi:

* state phức tạp
* nhiều action update
* nhiều component dùng chung state
* logic update lớn

([Reddit][2])

---

# 15. Khi nào KHÔNG nên dùng?

Không cần nếu:

* state nhỏ
* component đơn giản
* chỉ vài `useState`

---

# 16. Ưu điểm của mô hình này

| Ưu điểm             | Ý nghĩa                   |
| ------------------- | ------------------------- |
| Tách logic          | UI sạch hơn               |
| Tránh prop drilling | dễ maintain               |
| Dễ scale            | app lớn vẫn rõ ràng       |
| Centralized updates | mọi update đi qua reducer |
| Predictable         | dễ debug                  |

---

# 17. Nhược điểm

| Nhược điểm        | Giải thích                          |
| ----------------- | ----------------------------------- |
| Boilerplate nhiều | phải tạo reducer/context/actions    |
| Có thể overkill   | app nhỏ không cần                   |
| Re-render         | Context lớn có thể gây render nhiều |

---

# 18. Tư duy cốt lõi cần nhớ

## React muốn bạn chia:

```txt
State Logic
    ≠
State Distribution
```

---

## useReducer

giải quyết:

```txt
State thay đổi như thế nào?
```

---

## Context

giải quyết:

```txt
Ai có thể truy cập state?
```

---

# 19. Mô hình mental model chuẩn

```txt
Reducer
  = bộ não xử lý state

Dispatch
  = gửi yêu cầu thay đổi

Context
  = hệ thống phân phối state

Provider
  = trung tâm cấp dữ liệu

Custom Hook
  = API đẹp để dùng state
```

---

# 20. Recap ngắn gọn

## Quy trình chuẩn

```txt
1. Tạo reducer
2. useReducer()
3. Tạo context
4. Provider state + dispatch
5. useContext() ở component con
6. dispatch(action)
```

---

# 21. Ý quan trọng nhất của bài

React muốn bạn xây app theo hướng:

```txt
UI chỉ hiển thị
Logic state nằm riêng
```

Đây là tư duy cực kỳ quan trọng khi làm app React lớn. ([react.dev][3])

[1]: https://www.reddit.com/r/reactjs/comments/1cebqw7?utm_source=chatgpt.com "Scaling up React useReducer with Context"
[2]: https://www.reddit.com/r/reactjs/comments/1d2nt8u?utm_source=chatgpt.com "Do you feel like reducers and context go hand in hand?"
[3]: https://react.dev/learn/scaling-up-with-reducer-and-context?utm_source=chatgpt.com "Scaling Up with Reducer and Context – React"

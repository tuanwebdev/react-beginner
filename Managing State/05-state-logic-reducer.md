# Tách Logic State vào Reducer

> Hướng dẫn từ [react.dev/learn/extracting-state-logic-into-a-reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)

Khi component có nhiều event handler cùng cập nhật state theo nhiều cách khác nhau, code có thể trở nên khó đọc và dễ bug. **Reducer** giúp bạn gom toàn bộ logic cập nhật state vào một hàm duy nhất bên ngoài component.

---

## Vấn đề: Logic state rải rác khắp nơi

Xem component TaskApp — có 3 handler khác nhau đều gọi `setTasks`:

```jsx
export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([...tasks, { id: nextId++, text, done: false }]);
  }

  function handleChangeTask(task) {
    setTasks(tasks.map(t => t.id === task.id ? task : t));
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter(t => t.id !== taskId));
  }
  // ...
}
```

Khi component lớn dần → logic state rải rác → khó tìm, khó debug, dễ nhất quán sai. **Reducer** giải quyết điều này bằng cách gom tất cả vào một chỗ.

---

## Phép ẩn dụ: Reducer như một "bộ điều phối"

Hãy nghĩ như thế này:

```
Cách cũ (useState):
  Event handler → tự quyết định "làm gì với state" → gọi setTasks(...)

Cách mới (useReducer):
  Event handler → mô tả "việc gì vừa xảy ra" → dispatch({ type: 'added', ... })
                                                         ↓
                                              Reducer nhận action
                                                         ↓
                                              Reducer quyết định state mới
```

Event handler chỉ nói **"chuyện gì đã xảy ra"**, reducer quyết định **"state thay đổi như thế nào"**.

---

## 3 Bước chuyển từ useState sang useReducer

---

### Bước 1: Thay "set state" bằng "dispatch action"

Thay vì gọi `setTasks(...)`, gọi `dispatch({ type: '...', ...data })`:

```jsx
// TRƯỚC — nói React "làm gì"
function handleAddTask(text) {
  setTasks([...tasks, { id: nextId++, text, done: false }]);
}
function handleChangeTask(task) {
  setTasks(tasks.map(t => t.id === task.id ? task : t));
}
function handleDeleteTask(taskId) {
  setTasks(tasks.filter(t => t.id !== taskId));
}

// SAU — mô tả "việc gì xảy ra"
function handleAddTask(text) {
  dispatch({ type: 'added', id: nextId++, text });
}
function handleChangeTask(task) {
  dispatch({ type: 'changed', task });
}
function handleDeleteTask(taskId) {
  dispatch({ type: 'deleted', id: taskId });
}
```

**Action object** là object JavaScript thông thường mô tả sự kiện vừa xảy ra:

```js
// Quy ước: trường 'type' mô tả việc gì xảy ra (string)
// Các trường khác chứa dữ liệu cần thiết
{
  type: 'added',   // ← tên action, mô tả sự kiện
  id: 3,           // ← dữ liệu đi kèm
  text: 'Học React'
}
```

> 💡 **Quy ước đặt tên:** `type` thường mô tả **hành động của người dùng** theo dạng quá khứ: `'added'`, `'deleted'`, `'changed'`, `'reset_form'`... Không dùng tên kỹ thuật như `'set_tasks'`.

---

### Bước 2: Viết hàm reducer

Reducer là hàm nhận vào **(state hiện tại, action)** và trả về **state mới**:

```js
function yourReducer(state, action) {
  // Tính toán và return state mới
  // KHÔNG được mutation state trực tiếp
}
```

Viết reducer cho TaskApp dùng `switch/case`:

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {

    case 'added': {
      return [
        ...tasks,
        { id: action.id, text: action.text, done: false }
      ];
    }

    case 'changed': {
      return tasks.map(t =>
        t.id === action.task.id ? action.task : t
      );
    }

    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }

    default: {
      // Luôn throw ở default để phát hiện lỗi typo trong action type
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

> 💡 **Tại sao dùng switch thay vì if/else?** Switch dễ đọc hơn khi có nhiều case — nhìn vào ngay biết có bao nhiêu loại action. Nội dung thì như nhau.

> ⚠️ **Luôn bọc mỗi `case` trong `{}`** để tránh xung đột biến giữa các case.

---

### Bước 3: Dùng useReducer trong component

Thay `useState` bằng `useReducer`:

```jsx
import { useReducer } from 'react';

// Trước:
const [tasks, setTasks] = useState(initialTasks);

// Sau:
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
//     ↑          ↑                   ↑               ↑
//  state      dispatch fn         reducer fn      initial state
```

**Code hoàn chỉnh sau khi refactor:**

```jsx
import { useReducer } from 'react';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({ type: 'added', id: nextId++, text });
  }

  function handleChangeTask(task) {
    dispatch({ type: 'changed', task });
  }

  function handleDeleteTask(taskId) {
    dispatch({ type: 'deleted', id: taskId });
  }

  return (
    <>
      <h1>Lịch trình Prague</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

// Reducer đặt ngoài component (hoặc trong file riêng)
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, { id: action.id, text: action.text, done: false }];
    }
    case 'changed': {
      return tasks.map(t => t.id === action.task.id ? action.task : t);
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Thăm Kafka Museum', done: true },
  { id: 1, text: 'Xem múa rối', done: false },
  { id: 2, text: 'Chụp ảnh Lennon Wall', done: false },
];
```

**Mẹo:** Tách reducer ra file riêng `tasksReducer.js` để component gọn hơn:

```jsx
// tasksReducer.js
export default function tasksReducer(tasks, action) { ... }

// TaskApp.js
import tasksReducer from './tasksReducer.js';
```

---

## useReducer vs useState — Khi nào dùng cái nào?

| Tiêu chí | useState | useReducer |
|---|---|---|
| **Lượng code** | Ít hơn ban đầu | Nhiều hơn (cần viết reducer + actions) |
| **Dễ đọc** | Tốt khi state đơn giản | Tốt khi logic phức tạp — tách "what happened" khỏi "how to update" |
| **Debug** | Khó biết state được set ở đâu | Dễ: thêm `console.log` vào reducer, thấy mọi action + state change |
| **Testing** | Test kèm với component | Reducer là pure function → **test độc lập dễ dàng** |
| **Sở thích** | Phổ biến hơn | Một số dev thích cấu trúc rõ ràng hơn |

**Khi nào nên dùng useReducer:**

```
✅ Component có NHIỀU event handler cập nhật state theo nhiều cách
✅ Logic update phức tạp, liên quan đến nhiều trường
✅ Muốn test logic state độc lập với UI
✅ Đang gặp bug do state bị cập nhật sai và khó tìm nguồn
✅ Nhiều action dẫn đến cùng kiểu thay đổi (dễ đọc hơn với switch)

✅ useState vẫn tốt cho:
   - State đơn giản (boolean, số, string đơn)
   - Ít event handler
   - Logic update thẳng thắn
```

> 💡 Bạn có thể dùng cả `useState` và `useReducer` trong cùng một component — không cần chọn một.

---

## 2 Quy tắc viết Reducer tốt

### Quy tắc 1: Reducer phải thuần túy (Pure)

Giống render function, reducer **không được có side effect**:

```jsx
// ❌ SAI — side effects trong reducer
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      fetch('/api/tasks', { method: 'POST', body: JSON.stringify(action) }); // ❌ API call
      console.log('Task added!');  // ❌ console.log (side effect nhỏ nhưng vẫn tránh)
      tasks.push({ ... });         // ❌ mutation!
      return tasks;
    }
  }
}

// ✅ ĐÚNG — chỉ tính toán, không side effect, không mutation
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, { id: action.id, text: action.text, done: false }]; // tạo array mới
    }
  }
}
```

### Quy tắc 2: Mỗi action mô tả một tương tác của người dùng

```jsx
// ❌ SAI — dispatch nhiều action riêng lẻ cho một hành động của user
function handleReset() {
  dispatch({ type: 'set_name', value: '' });
  dispatch({ type: 'set_email', value: '' });
  dispatch({ type: 'set_phone', value: '' });
  dispatch({ type: 'set_address', value: '' });
  dispatch({ type: 'set_message', value: '' });
}

// ✅ ĐÚNG — một action mô tả một hành động ("user nhấn Reset")
function handleReset() {
  dispatch({ type: 'reset_form' });
}

// Reducer xử lý tất cả trong một case:
case 'reset_form': {
  return { name: '', email: '', phone: '', address: '', message: '' };
}
```

Lợi ích: Log action dễ đọc, dễ debug — thấy ngay "user đã làm gì" theo từng bước.

---

## Dùng Immer để viết Reducer gọn hơn

Reducer phải tránh mutation nhưng đôi khi spread lồng nhiều tầng rất dài dòng. Immer cho phép viết như mutation trong reducer:

```bash
npm install use-immer
```

```jsx
import { useImmerReducer } from 'use-immer';

function tasksReducer(draft, action) {
  // Với Immer: có thể mutation draft trực tiếp!
  switch (action.type) {
    case 'added': {
      draft.push({ id: action.id, text: action.text, done: false });
      // push() được phép vì draft là Proxy của Immer
      break;  // dùng break thay vì return vì Immer tự tạo state mới
    }
    case 'changed': {
      const index = draft.findIndex(t => t.id === action.task.id);
      draft[index] = action.task;  // gán trực tiếp được!
      break;
    }
    case 'deleted': {
      return draft.filter(t => t.id !== action.id);
      // có thể return hoặc mutation tùy loại thao tác
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);
  // ... event handlers và JSX giữ nguyên
}
```

**So sánh:**

```jsx
// Không Immer — phải tạo array/object mới
case 'added': {
  return [...tasks, { id: action.id, text: action.text, done: false }];
}

// Có Immer — ngắn gọn hơn
case 'added': {
  draft.push({ id: action.id, text: action.text, done: false });
  break;
}
```

---

## Tại sao gọi là "Reducer"?

Tên gọi xuất phát từ hàm `Array.reduce()`:

```js
// Array.reduce: tích lũy mảng thành một giá trị
const sum = [1, 2, 3, 4, 5].reduce(
  (result, number) => result + number,
  0  // initial value
);
// Mỗi bước: result = kết quả trước, number = phần tử hiện tại

// React Reducer: tương tự!
// state = kết quả trước, action = "phần tử" hiện tại
function reducer(state, action) {
  return nextState;
}

// Thậm chí có thể dùng Array.reduce để tính final state:
const finalState = actions.reduce(tasksReducer, initialState);
// [action1, action2, action3] → state cuối cùng
```

---

## Cấu trúc file đề xuất

Khi dùng reducer, nên tổ chức file như sau:

```
src/
├── App.js
├── TaskApp.js          ← component (chỉ UI + dispatch)
├── tasksReducer.js     ← reducer function (logic state)
├── AddTask.js
└── TaskList.js
```

```jsx
// tasksReducer.js — chỉ chứa logic, không có UI
export const initialTasks = [
  { id: 0, text: 'Thăm Kafka Museum', done: true },
  { id: 1, text: 'Xem múa rối', done: false },
];

export function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added':   { return [...tasks, { id: action.id, text: action.text, done: false }]; }
    case 'changed': { return tasks.map(t => t.id === action.task.id ? action.task : t); }
    case 'deleted': { return tasks.filter(t => t.id !== action.id); }
    default:        { throw Error('Unknown action: ' + action.type); }
  }
}
```

```jsx
// TaskApp.js — chỉ chứa UI + dispatch, không có logic state
import { useReducer } from 'react';
import { tasksReducer, initialTasks } from './tasksReducer.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
}
```

---

## Tóm tắt

```
Reducer là gì:
  Hàm (state, action) → nextState
  Gom toàn bộ logic cập nhật state vào một nơi

3 bước chuyển useState → useReducer:
  1. dispatch({ type: 'action_name', ...data })  thay vì setX(...)
  2. Viết reducer function với switch/case
  3. Thay useState bằng useReducer(reducer, initialState)

Action object:
  { type: 'added', id: 1, text: 'Học React' }
  → type mô tả việc xảy ra (quá khứ)
  → Các trường khác là dữ liệu cần thiết

2 quy tắc viết reducer tốt:
  1. THUẦN TÚY: không side effect, không mutation
  2. MỖI ACTION = MỘT TƯƠNG TÁC người dùng (không dispatch nhiều lần)

Immer: dùng useImmerReducer để viết như mutation, ngắn gọn hơn

Khi nào dùng useReducer:
  → Nhiều handler cập nhật cùng state theo nhiều cách
  → Logic phức tạp, khó debug
  → Muốn test state logic độc lập
```

---

## Bài tập tự luyện

### Bài 1: Thêm action 'toggle' vào reducer

```jsx
// Reducer hiện tại chỉ có 'added' và 'deleted'
// Thêm case 'toggle' để bật/tắt trường done của task
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, { id: action.id, text: action.text, done: false }];
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    // TODO: thêm case 'toggle'
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, { id: action.id, text: action.text, done: false }];
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    case 'toggle': {
      return tasks.map(t =>
        t.id === action.id ? { ...t, done: !t.done } : t
      );
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

// Dispatch từ event handler:
function handleToggle(taskId) {
  dispatch({ type: 'toggle', id: taskId });
}
```
</details>

---

### Bài 2: Test reducer độc lập

Viết test đơn giản cho tasksReducer (không cần framework):

```jsx
// Hãy viết 3 test case cho tasksReducer
const initialTasks = [
  { id: 0, text: 'Task A', done: false },
  { id: 1, text: 'Task B', done: false },
];
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
// Test 1: thêm task
const afterAdd = tasksReducer(initialTasks, {
  type: 'added', id: 2, text: 'Task C'
});
console.assert(afterAdd.length === 3, 'Should have 3 tasks');
console.assert(afterAdd[2].text === 'Task C', 'New task text correct');
console.assert(afterAdd[2].done === false, 'New task not done');

// Test 2: xóa task
const afterDelete = tasksReducer(initialTasks, {
  type: 'deleted', id: 0
});
console.assert(afterDelete.length === 1, 'Should have 1 task');
console.assert(afterDelete[0].id === 1, 'Correct task remains');

// Test 3: thay đổi task
const afterChange = tasksReducer(initialTasks, {
  type: 'changed', task: { id: 0, text: 'Task A updated', done: true }
});
console.assert(afterChange[0].text === 'Task A updated', 'Text updated');
console.assert(afterChange[0].done === true, 'Done updated');
```

Đây là ưu điểm lớn của reducer: **test thuần túy, không cần render component!**
</details>

---

### Bài 3: Sửa lỗi reducer

```jsx
// Reducer này có lỗi — tìm và sửa
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      state.count++;         // ← lỗi 1
      return state;
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    // ← lỗi 2: thiếu default
  }
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment': {
      return { count: state.count + 1 };  // ✅ tạo object mới, không mutation
    }
    case 'decrement': {
      return { count: state.count - 1 };
    }
    case 'reset': {
      return { count: 0 };
    }
    default: {
      throw Error('Unknown action: ' + action.type);  // ✅ thêm default
    }
  }
}
```

**2 lỗi:**
1. `state.count++` — mutation trực tiếp. Phải tạo object mới: `{ count: state.count + 1 }`
2. Thiếu `default` case — nếu dispatch action không tồn tại, reducer sẽ trả về `undefined` âm thầm gây bug khó tìm
</details>

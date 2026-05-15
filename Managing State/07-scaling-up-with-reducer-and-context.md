Tất nhiên rồi, dưới đây là toàn bộ nội dung bài viết "Scaling Up with Reducer and Context" từ React.dev được trình bày dưới dạng markdown:

---

# Mở rộng với Reducer và Context

Reducer cho phép bạn hợp nhất logic cập nhật state của một component. Context cho phép bạn truyền thông tin sâu xuống các component khác. Bạn có thể kết hợp reducer và context cùng nhau để quản lý state của một màn hình phức tạp.

### Bạn sẽ học được

- Cách kết hợp reducer với context
- Cách tránh truyền state và dispatch qua props
- Cách giữ logic context và state trong một tệp riêng biệt

## Kết hợp reducer với context

Trong ví dụ này từ phần giới thiệu về reducer, state được quản lý bởi một reducer. Hàm reducer chứa tất cả logic cập nhật state và được khai báo ở cuối tệp này. Một reducer giúp các event handler ngắn gọn và súc tích. Tuy nhiên, khi ứng dụng của bạn phát triển, bạn có thể gặp một khó khăn khác. Hiện tại, state `tasks` và hàm `dispatch` chỉ khả dụng trong component `TaskApp` cấp cao nhất. Để các component khác có thể đọc danh sách công việc hoặc thay đổi nó, bạn phải truyền xuống một cách tường minh state hiện tại và các event handler (dùng để thay đổi state đó) dưới dạng props.

Ví dụ: `TaskApp` truyền danh sách công việc và các event handler cho `TaskList`:
```jsx
<TaskList
  tasks={tasks}
  onChangeTask={handleChangeTask}
  onDeleteTask={handleDeleteTask}
/>
```
Và `TaskList` truyền các event handler cho `Task`:
```jsx
<Task
  task={task}
  onChange={onChangeTask}
  onDelete={onDeleteTask}
/>
```
Trong một ví dụ nhỏ như thế này, cách làm này hoạt động tốt, nhưng nếu bạn có hàng chục hoặc hàng trăm component ở giữa, việc truyền tất cả state và hàm xuống có thể khá bực bội!

Đây là lý do tại sao, như một giải pháp thay thế cho việc truyền chúng qua props, bạn có thể muốn đặt cả state `tasks` và hàm `dispatch` vào trong context. **Bằng cách này, bất kỳ component nào bên dưới `TaskApp` trong cây đều có thể đọc `tasks` và gửi các action mà không cần "khoan prop" lặp đi lặp lại.**

Dưới đây là cách bạn có thể kết hợp reducer với context:

1.  **Tạo context.**
2.  **Đặt state và dispatch vào context.**
3.  **Sử dụng context ở bất kỳ đâu trong cây.**

### Bước 1: Tạo context

Hook `useReducer` trả về `tasks` hiện tại và hàm `dispatch` cho phép bạn cập nhật chúng:
```js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```
Để truyền chúng xuống cây, bạn sẽ tạo hai context riêng biệt:
*   `TasksContext` cung cấp danh sách công việc hiện tại.
*   `TasksDispatchContext` cung cấp hàm cho phép các component gửi action.

Xuất chúng từ một tệp riêng để bạn có thể import chúng từ các tệp khác sau này:
```js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```
Ở đây, bạn đang truyền `null` làm giá trị mặc định cho cả hai context. Các giá trị thực tế sẽ được cung cấp bởi component `TaskApp`.

### Bước 2: Đặt state và dispatch vào context

Bây giờ bạn có thể import cả hai context trong component `TaskApp` của mình. Lấy `tasks` và `dispatch` được trả về bởi `useReducer()` và cung cấp chúng cho toàn bộ cây bên dưới:
```jsx
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        ...
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```
*(Lưu ý: Trong code gốc có sử dụng cú pháp rút gọn `<TasksContext value={tasks}>`, nhưng về bản chất nó là `<TasksContext.Provider value={tasks}>`)*. Hiện tại, bạn vẫn truyền thông tin qua cả props và context. Trong bước tiếp theo, bạn sẽ loại bỏ việc truyền prop.

### Bước 3: Sử dụng context ở bất kỳ đâu trong cây

Bây giờ bạn không cần truyền danh sách công việc hoặc các event handler xuống cây nữa:
```jsx
<TasksContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    <h1>Day off in Kyoto</h1>
    <AddTask />
    <TaskList />
  </TasksDispatchContext.Provider>
</TasksContext.Provider>
```
Thay vào đó, bất kỳ component nào cần danh sách công việc đều có thể đọc nó từ `TasksContext`:
```js
export default function TaskList() {
  const tasks = useContext(TasksContext);
  // ...
```
Để cập nhật danh sách công việc, bất kỳ component nào cũng có thể đọc hàm `dispatch` từ context và gọi nó:
```jsx
export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Add</button>
    // ...
```
Component `TaskApp` không truyền bất kỳ event handler nào xuống, và `TaskList` cũng không truyền event handler nào cho component `Task` nữa. Mỗi component tự đọc context mà nó cần. State vẫn "sống" trong component `TaskApp` cấp cao nhất, được quản lý bằng `useReducer`. Nhưng `tasks` và `dispatch` của nó giờ đã có sẵn cho mọi component bên dưới trong cây bằng cách import và sử dụng các context này.

## Di chuyển tất cả kết nối vào một tệp duy nhất

Bạn không bắt buộc phải làm điều này, nhưng bạn có thể tiếp tục dọn dẹp các component bằng cách di chuyển cả reducer và context vào một tệp duy nhất. Hiện tại, `TasksContext.js` chỉ chứa hai khai báo context. Tệp này sắp trở nên "đông đúc" hơn! Bạn sẽ di chuyển reducer vào cùng tệp đó. Sau đó, bạn sẽ khai báo một component `TasksProvider` mới trong cùng tệp. Component này sẽ gắn kết tất cả các mảnh lại với nhau:

1.  Nó sẽ quản lý state với một reducer.
2.  Nó sẽ cung cấp cả hai context cho các component bên dưới.
3.  Nó sẽ nhận `children` như một prop để bạn có thể truyền JSX vào đó.

```jsx
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```
**Điều này loại bỏ tất cả sự phức tạp và kết nối khỏi component `TaskApp` của bạn:**
```jsx
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```
Bạn cũng có thể xuất các hàm sử dụng context từ `TasksContext.js`:
```js
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```
Khi một component cần đọc context, nó có thể làm điều đó thông qua các hàm này:
```js
const tasks = useTasks();
const dispatch = useTasksDispatch();
```
Điều này không thay đổi hành vi theo bất kỳ cách nào, nhưng nó cho phép bạn sau này tách các context này sâu hơn hoặc thêm một số logic vào các hàm này. **Giờ đây, tất cả kết nối context và reducer đều nằm trong `TasksContext.js`. Điều này giữ cho các component sạch sẽ và gọn gàng, tập trung vào những gì chúng hiển thị hơn là nơi chúng lấy dữ liệu.**

Bạn có thể coi `TasksProvider` như một phần của màn hình biết cách xử lý các công việc, `useTasks` như một cách để đọc chúng và `useTasksDispatch` như một cách để cập nhật chúng từ bất kỳ component nào bên dưới trong cây.

### Lưu ý

Các hàm như `useTasks` và `useTasksDispatch` được gọi là **Custom Hooks**. Hàm của bạn được coi là custom Hook nếu tên của nó bắt đầu bằng `use`. Điều này cho phép bạn sử dụng các Hook khác, như `useContext`, bên trong nó.

Khi ứng dụng của bạn phát triển, bạn có thể có nhiều cặp context-reducer như thế này. Đây là một cách mạnh mẽ để mở rộng ứng dụng của bạn và **nâng state lên** mà không cần quá nhiều công sức mỗi khi bạn muốn truy cập dữ liệu ở sâu trong cây.

## Tóm tắt

- Bạn có thể kết hợp reducer với context để cho phép bất kỳ component nào đọc và cập nhật state phía trên nó.
- Để cung cấp state và hàm dispatch cho các component bên dưới:
    - Tạo hai context (cho state và cho hàm dispatch).
    - Cung cấp cả hai context từ component sử dụng reducer.
    - Sử dụng một trong hai context từ các component cần đọc chúng.
- Bạn có thể dọn dẹp các component hơn nữa bằng cách di chuyển tất cả kết nối vào một tệp.
    - Bạn có thể xuất một component như `TasksProvider` để cung cấp context.
    - Bạn cũng có thể xuất các custom Hook như `useTasks` và `useTasksDispatch` để đọc nó.
- Bạn có thể có nhiều cặp context-reducer như thế này trong ứng dụng của mình.
---
title: NextJS
draft: false
tags:
---
 
# 介绍
# 安装
```sh
npx create-next-app@latest
```
# 概念
**React 假设你编写的所有组件都是纯函数**。也就是说，对于相同的输入，你所编写的 React 组件必须总是返回相同的 JSX。
## 组件
```ts
function MyButton() {
  return (
    <button>
      我是一个按钮
    </button>
  );
}

export default function MyApp() {
  return (
    <div>
      <h1>欢迎来到我的应用</h1>
      <MyButton />
    </div>
  );
}

```
jsx是在js中使用html标签的一种语法。
### 条件渲染
```ts
# 方式 1 if
let content;  
if (isLoggedIn) {  
	content = <AdminPanel />;  
} else {  
	content = <LoginForm />;  
}  
	return (  
		<div>  
		{content}  
		</div>  
	);
# 方式 2 三元运算符
<div>  
	{isLoggedIn ? (  
		<AdminPanel />  
	) : (  
		<LoginForm />  
	)}  
</div>

# 方式 3 逻辑与
<div>  
	{isLoggedIn && <AdminPanel />}  
</div>
```
### 列表渲染
```ts
const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];

const listItems = products.map(product =>
  <li key={product.id}>
    {product.title}
  </li>
);

return (
  <ul>{listItems}</ul>
);
```
### 样式
```ts
<img className="avatar" />
```

css文件
```css
.avatar {  
	border-radius: 50%;  
}
```
### 事件
```ts
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      点我
    </button>
  );
}
```
### state && props
```ts
# props
export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src="url"
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

# state
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>共同更新的计数器</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}
```
## 状态管理
### Reducer
reducer 函数就是你放置状态逻辑的地方。它接受两个参数，分别为当前 state 和 action 对象，并且返回的是更新后的 state：
```ts
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知 action：' + action.type);
    }
  }
}

```

使用 reducer ，先导入 userReducer，然后 `const [tasks, dispatch] = useReducer(tasksReducer, initialTasks)`，useReducer 钩子接受 2 个参数：一个 reducer 函数
一个初始的 state，它返回如下内容：一个有状态的值，一个 dispatch 函数（用来 “派发” 用户操作给 reducer）
```ts
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import tasksReducer from './tasksReducer.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>布拉格的行程安排</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: '参观卡夫卡博物馆', done: true},
  {id: 1, text: '看木偶戏', done: false},
  {id: 2, text: '打卡列侬墙', done: false},
];


```
使用 Immer 简化 reducer 
与在平常的 state 中 修改对象 和 数组 一样，可以使用 Immer 这个库来简化 reducer。在这里，useImmerReducer 让你可以通过 push 或 arr[i] = 来修改 state ：
```ts
import { useImmerReducer } from 'use-immer';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知 action：' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>布拉格的行程安排</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: '参观卡夫卡博物馆', done: true},
  {id: 1, text: '看木偶戏', done: false},
  {id: 2, text: '打卡列侬墙', done: false},
];

```
### Context
Context 让父组件可以为它下面的整个组件树提供数据。
创建 context
```ts
import { createContext } from 'react';

export const LevelContext = createContext(1);
```
使用 context
```ts
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext value={level}>
        {children}
      </LevelContext>
    </section>
  );
}
```

## Hook
## Layout && Page
layout 是通用的UI 模板，Page 是路由后展示的页面
## Linking && Navigating
可以动态渲染页面，不是一次性整个加载页面
## Server && Client Components
CSR
河马下单-》自己做饭，每个人做饭速度不一样

SSR
直接给饭店下单
ESR
预制菜

## Partial Prerendering
## Fetching && Updating Data
```typescript
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}

```
## Error Handing
```typescript
'use client' // Error boundaries must be Client Components
 
import { useEffect } from 'react'
 
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error(error)
  }, [error])
 
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </button>
    </div>
  )
}
```
## CSS
- css module
- global css
- saas
## Route Handlers && Middleware
app 路由和中间件

## Deploying


# UI 布局
先自己构想出容器的分布，一共多少个容器，分别怎样放置，用什么布局方式。然后再在每个容器中放置元素。
## 布局方式
一些场景
- 千人一面
- 千人多面
- 千人千面
## 配色
## 数据显示
## 交互
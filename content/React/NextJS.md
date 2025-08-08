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
## JSX
在js中使用html标签的一种语法。
## Hook
## Layout && Page
layout 是通用的UI 模板，Page 是路由后展示的页面
## Linking && Navigating
可以动态渲染页面，不是一次性整个加载页面
## Server && Client Components

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
## 配色
## 数据显示
## 交互
---
title: "JavaScript入门：Web开发必备"
date: 2026-06-13
draft: false
tags: ["javascript", "web开发", "前端"]
categories: ["Web开发"]
author: "cmj156"
summary: "JavaScript是Web开发的核心语言，本文将带你了解JavaScript的基础知识。"
image: "https://picsum.photos/seed/javascript/800/400"
---

## JavaScript简介

JavaScript是一种轻量级、解释型的编程语言，主要用于Web开发。它是现代Web应用的核心技术之一。

## 基础语法

### 1. 变量声明

```javascript
// 使用let和const声明变量
let age = 25;
const PI = 3.14159;

// 传统方式（不推荐）
var name = "张三";
```

### 2. 数据类型

```javascript
// 基本类型
let string = "Hello";
let number = 42;
let boolean = true;
let nullValue = null;
let undefinedValue = undefined;

// 引用类型
let array = [1, 2, 3];
let object = {
    name: "李四",
    age: 30
};
```

### 3. 函数

```javascript
// 函数声明
function greet(name) {
    return `你好, ${name}!`;
}

// 箭头函数
const add = (a, b) => a + b;

// 调用函数
console.log(greet("小明"));
console.log(add(5, 3));
```

### 4. 条件语句

```javascript
let score = 85;

if (score >= 90) {
    console.log("优秀");
} else if (score >= 80) {
    console.log("良好");
} else if (score >= 60) {
    console.log("及格");
} else {
    console.log("不及格");
}
```

### 5. 循环

```javascript
// for循环
for (let i = 0; i < 5; i++) {
    console.log(`数字: ${i}`);
}

// while循环
let count = 0;
while (count < 3) {
    console.log(`计数: ${count}`);
    count++;
}
```

## DOM操作

JavaScript可以操作HTML文档：

```javascript
// 获取元素
const element = document.getElementById("myElement");

// 修改内容
element.textContent = "新的内容";

// 修改样式
element.style.color = "red";
element.style.fontSize = "20px";
```

## 事件处理

```javascript
// 添加事件监听器
document.getElementById("myButton").addEventListener("click", function() {
    alert("按钮被点击了！");
});
```

## 异步编程

```javascript
// Promise
function fetchData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve("数据获取成功！");
        }, 1000);
    });
}

// 使用async/await
async function getData() {
    try {
        const data = await fetchData();
        console.log(data);
    } catch (error) {
        console.error("错误:", error);
    }
}
```

## 总结

JavaScript是Web开发的核心语言。通过本文的学习，你已经掌握了JavaScript的基础知识。继续实践，你会成为JavaScript专家！

---

**上一篇**：[Python基础教程](/posts/python-basics/)  
**下一篇**：[CSS布局技巧](/posts/css-layout/)

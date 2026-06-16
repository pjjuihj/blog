---
title: "Python基础教程：快速入门"
date: 2026-06-14
draft: false
tags: ["python", "编程语言", "入门"]
categories: ["编程教程"]
author: "cmj156"
comments: true
summary: "Python是当前最流行的编程语言之一，本文将带你快速入门Python编程。"
image: "https://picsum.photos/seed/python/800/400"
---

## 为什么学习Python？

Python以其简洁的语法和强大的功能而闻名，是初学者和专业开发者的首选语言。

## Python基础语法

### 1. 变量和数据类型

```python
# 变量赋值
name = "张三"
age = 25
height = 1.75
is_student = True

# 打印变量
print(f"姓名: {name}, 年龄: {age}")
```

### 2. 条件语句

```python
score = 85

if score >= 90:
    print("优秀")
elif score >= 80:
    print("良好")
elif score >= 60:
    print("及格")
else:
    print("不及格")
```

### 3. 循环语句

```python
# for循环
for i in range(5):
    print(f"数字: {i}")

# while循环
count = 0
while count < 3:
    print(f"计数: {count}")
    count += 1
```

### 4. 函数定义

```python
def greet(name, greeting="你好"):
    """问候函数"""
    return f"{greeting}, {name}!"

# 调用函数
message = greet("小明")
print(message)
```

### 5. 列表和字典

```python
# 列表
fruits = ["苹果", "香蕉", "橙子"]
fruits.append("葡萄")

# 字典
person = {
    "name": "李四",
    "age": 30,
    "city": "北京"
}
```

## 实战练习

让我们创建一个简单的计算器程序：

```python
def calculator(num1, num2, operation):
    if operation == '+':
        return num1 + num2
    elif operation == '-':
        return num1 - num2
    elif operation == '*':
        return num1 * num2
    elif operation == '/':
        return num1 / num2 if num2 != 0 else "错误：除数不能为零"
    else:
        return "未知操作"

# 测试计算器
result = calculator(10, 5, '+')
print(f"10 + 5 = {result}")
```

## 总结

Python是一门简单而强大的编程语言。通过本文的学习，你已经掌握了Python的基础语法。继续练习，你会成为Python高手！


---
title: "STM32 MPU6050 云台控制器：从零到实战"
date: 2026-06-16
draft: false
tags: ["stm32", "mpu6050", "freertos", "pid", "云台", "嵌入式", "dmp"]
categories: ["嵌入式开发"]
author: "pujj"
comments: true
summary: "分享我的 STM32 + MPU6050 DMP 双轴云台控制项目，包含 FreeRTOS 多任务架构、PID 控制、蓝牙通信等完整实现。"
image: "https://picsum.photos/seed/gimbal/800/400"
---

## 项目简介

这是一个基于 **STM32F407 + MPU6050 DMP** 的双轴云台控制系统。使用 MPU6050 的 DMP（数字运动处理器）进行硬件姿态解算，通过 FreeRTOS 实现多任务调度，控制双轴舵机云台稳定跟随目标角度。

**项目地址：** [GitHub - stm32-mpu6050-gimbal](https://github.com/pjjuihj/stm32-mpu6050-gimbal)

## 硬件平台

| 组件 | 型号 | 说明 |
|------|------|------|
| MCU | STM32F407VETx | Cortex-M4F, 168MHz |
| 传感器 | MPU6050 | 6 轴陀螺仪 + 加速度计 |
| 显示 | 0.96" OLED | I2C 接口 |
| 舵机 | SG90/MG90S | PWM 控制 |
| 通信 | HC-05 蓝牙 | UART 串口 |

## 系统架构

### FreeRTOS 多任务设计

系统采用 FreeRTOS 实时操作系统，将功能分解为 6 个独立任务：

```
┌─────────────────────────────────────────────────────────────┐
│                      数据流                                  │
│  MPU6050 → MPU_Task → att_queue → PID_Task → pwm_queue → PWM_Task → 舵机
└─────────────────────────────────────────────────────────────┘
```

| 任务 | 优先级 | 周期 | 职责 |
|------|--------|------|------|
| MPU_Task | 高 (3) | 10ms | 读取 MPU6050 DMP 姿态数据 |
| PID_Task | 高 (3) | 10ms | PID 控制计算 |
| PWM_Task | 中 (2) | 阻塞 | 舵机 PWM 输出 |
| BT_Task | 中 (2) | 阻塞 | 蓝牙命令解析 |
| OLED_Task | 低 (1) | 200ms | OLED 显示刷新 |
| LED_Task | 低 (1) | 200ms | 状态指示灯 |

### 核心代码

**任务创建（freertos.c）：**

```c
/* 创建队列 */
att_queue = xQueueCreate(ATT_QUEUE_LENGTH, sizeof(Attitude_t));
pwm_queue = xQueueCreate(PWM_QUEUE_LENGTH, sizeof(PWM_t));
bt_queue  = xQueueCreate(BT_QUEUE_LENGTH, BT_BUF_SIZE);

/* 创建任务 */
xTaskCreate(MPU_Task,  "MPU",  MPU_TASK_STACK,  NULL, MPU_TASK_PRIORITY,  NULL);
xTaskCreate(PID_Task,  "PID",  PID_TASK_STACK,  NULL, PID_TASK_PRIORITY,  NULL);
xTaskCreate(PWM_Task,  "PWM",  PWM_TASK_STACK,  NULL, PWM_TASK_PRIORITY,  NULL);
xTaskCreate(OLED_Task, "OLED", OLED_TASK_STACK, NULL, OLED_TASK_PRIORITY, NULL);
xTaskCreate(BT_Task,   "BT",   BT_TASK_STACK,   NULL, BT_TASK_PRIORITY,   NULL);
xTaskCreate(LED_Task,  "LED",  LED_TASK_STACK,  NULL, LED_TASK_PRIORITY,  NULL);
```

## MPU6050 DMP 姿态解算

### 什么是 DMP？

DMP（Digital Motion Processor）是 MPU6050 内置的硬件姿态解算引擎。它可以直接在传感器内部完成四元数到欧拉角的转换，大大减轻主控 MCU 的计算负担。

### 适配器层设计

为了支持不同的 DMP 驱动库，我设计了一个统一的适配器层：

```c
// mpu6050_adapter.h
typedef struct {
    float pitch;    /* 俯仰角 (°) */
    float roll;     /* 横滚角 (°) */
    float yaw;      /* 偏航角 (°) */
} Attitude_t;

int MPU_Adapter_Init(void);
uint8_t MPU_Adapter_GetData(Attitude_t *att);
const char* MPU_Adapter_GetDriverName(void);
```

### I2C 总线恢复机制

I2C 总线在实际应用中可能会挂死，我实现了一个自动恢复机制：

```c
void I2C_Bus_Recovery(void)
{
    HAL_I2C_DeInit(&hi2c1);
    vTaskDelay(pdMS_TO_TICKS(10));
    MX_I2C1_Init();
    vTaskDelay(pdMS_TO_TICKS(10));
}

// 在 MPU_Adapter_GetData 中自动触发
if (fail_count >= MPU_FAIL_THRESHOLD) {
    I2C_Bus_Recovery();
    fail_count = 0;
}
```

## PID 云台控制

### 控制模式

系统支持两种控制模式：

1. **PID 闭环模式**：精确的角度控制
2. **直接跟随模式**：传感器角度直接映射到舵机

```c
if (gimbal.mode == GIMBAL_MODE_PID) {
    /* PID 闭环模式 */
    gimbal.pid_pitch.target = gimbal.target_pitch;
    float out_p = PID_Calculate(&gimbal.pid_pitch, att.pitch);
    pwm.pitch = PITCH_PWM_CENTER + (int16_t)out_p;
} else {
    /* 直接跟随模式 */
    float pitch_clamped = att.pitch;
    if (fabsf(pitch_clamped) < SERVO_DEAD_ZONE) pitch_clamped = 0.0f;
    pwm.pitch = (int16_t)(PITCH_PWM_CENTER + pitch_clamped * (PWM_RANGE / PITCH_ANGLE_RANGE));
}
```

### 舵机保护

```c
/* 限幅保护舵机，防止过载 */
if (pwm.pitch < SERVO_PWM_MIN) pwm.pitch = SERVO_PWM_MIN;
if (pwm.pitch > SERVO_PWM_MAX) pwm.pitch = SERVO_PWM_MAX;
```

## 蓝牙串口控制

支持通过蓝牙远程调整 PID 参数和控制云台：

| 命令 | 功能 | 示例 |
|------|------|------|
| `T` | 读取当前角度 | `T\r` |
| `P<value>` | 设置 Kp | `P1.5\r` |
| `I<value>` | 设置 Ki | `I0.01\r` |
| `D<value>` | 设置 Kd | `D0.5\r` |
| `S<angle>` | 设置目标角度 | `S30\r` |
| `G0/G1` | 切换控制模式 | `G1\r` |

## OLED 多页面显示

系统实现了 3 个显示页面，通过按键或蓝牙切换：

1. **主页面**：显示当前角度、目标角度、PWM 输出
2. **PID 页面**：显示 Kp/Ki/Kd 参数
3. **系统页面**：显示控制模式、运行时间、失败计数

```c
void OLED_DrawPage(OLED_Page_t page, char *buf, size_t len)
{
    switch (page) {
        case OLED_PAGE_MAIN:
            snprintf(buf, len, "P:%.1f->%.1f   ", gimbal.current_pitch, gimbal.target_pitch);
            OLED_ShowString(0, 0, (uint8_t *)buf, OLED_FONT_SIZE, 1);
            // ...
            break;
        case OLED_PAGE_PID:
            snprintf(buf, len, "Kp:%.2f      ", gimbal.pid_pitch.Kp);
            OLED_ShowString(0, 0, (uint8_t *)buf, OLED_FONT_SIZE, 1);
            // ...
            break;
    }
}
```

## 代码质量优化

经过专业的代码审查和优化，项目代码质量达到 **9.5/10**：

### 主要优化点

| 优化项 | 说明 |
|--------|------|
| 常量定义 | 所有 magic number 定义为命名常量 |
| 错误处理 | 统一错误码，添加 configASSERT 检查 |
| 代码复用 | I2C 恢复逻辑统一为单一函数 |
| 注释完善 | 详细的函数注释和调用说明 |
| 栈安全 | FreeRTOS 任务栈增大到安全值 |

### 项目文件结构

```
stm32-mpu6050-gimbal/
├── Board/
│   ├── Gimbal/          # 云台 PID 控制
│   ├── MPU6050/         # MPU6050 DMP 驱动
│   └── OLED/            # OLED 显示驱动
├── Core/
│   ├── Inc/app_tasks.h  # 任务定义和常量
│   └── Src/
│       ├── main.c       # 主程序
│       ├── freertos.c   # FreeRTOS 初始化
│       └── app_tasks.c  # 任务实现
└── MDK-ARM/             # Keil 工程
```

## 开发过程中的坑

### 1. I2C 总线冲突

MPU6050 和 OLED 共享 I2C 总线，需要合理处理总线仲裁。解决方案：使用互斥锁保护 I2C 访问。

### 2. FreeRTOS 栈溢出

初始任务栈设置过小（512B），导致运行时栈溢出。解决方案：增大栈大小到 1KB+，并启用栈溢出检测。

### 3. DMP 初始化失败

MPU6050 DMP 初始化有时会失败。解决方案：添加重试机制和 I2C 总线恢复。

## 总结

这个项目让我深入学习了：

- **MPU6050 DMP** 硬件姿态解算
- **FreeRTOS** 多任务架构设计
- **PID 控制** 算法实现
- **I2C/SPI/UART** 通信协议
- **代码质量** 审查和优化

项目已开源在 GitHub，欢迎 Star 和交流！

**项目地址：** [https://github.com/pjjuihj/stm32-mpu6050-gimbal](https://github.com/pjjuihj/stm32-mpu6050-gimbal)

---

如果你对嵌入式开发感兴趣，欢迎关注我的博客，后续会分享更多 STM32 项目实战经验！

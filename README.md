# STM32 双电机 PWM 驱动

基于 **STM32F103RCT6 + HAL 库** 的双直流电机驱动代码，使用 **TIM4 四路 PWM** 控制左右电机，可实现前进、后退、原地转向、差速转弯、制动和滑行。

## 硬件配置

| 引脚 | TIM4 通道 | 功能 |
|---|---|---|
| PB6 | CH1 | 左电机前进 PWM |
| PB7 | CH2 | 左电机后退 PWM |
| PB8 | CH3 | 右电机前进 PWM |
| PB9 | CH4 | 右电机后退 PWM |

TIM4 配置：

```c
Prescaler = 63;
Period = 99;
```

PWM 频率约为 **10 kHz**，速度参数建议范围为 **0~99**。

## 主要功能

```c
Car_Forward(speed);                 // 前进
Car_Backward(speed);                // 后退
Car_RotateLeft(speed);              // 原地左转
Car_RotateRight(speed);             // 原地右转

Car_ForwardLeft(low, high);         // 前进左转
Car_ForwardRight(low, high);        // 前进右转
Car_BackwardLeft(low, high);        // 后退左转
Car_BackwardRight(low, high);       // 后退右转

Car_Brake();                        // 制动
Car_Slide();                        // 滑行
```

## 使用示例

启动 TIM4 四路 PWM：

```c
HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_1);
HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_2);
HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_3);
HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_4);
```

控制小车：

```c
Car_Forward(60);
HAL_Delay(1000);

Car_Slide();
HAL_Delay(300);

Car_RotateLeft(50);
HAL_Delay(500);

Car_Slide();
```

## 控制逻辑

| 动作 | CH1 | CH2 | CH3 | CH4 |
|---|---:|---:|---:|---:|
| 前进 | speed | 0 | speed | 0 |
| 后退 | 0 | speed | 0 | speed |
| 原地左转 | 0 | speed | speed | 0 |
| 原地右转 | speed | 0 | 0 | speed |
| 滑行 | 0 | 0 | 0 | 0 |

差速转弯通过设置左右电机不同 PWM 占空比实现。

例如：

```c
Car_ForwardLeft(30, 70);
```

左轮 PWM 为 30%，右轮 PWM 为 70%，车辆向左转弯。

## 工程文件

```text
Core/
├── Inc/
│   └── Car_Drive.h
└── Src/
    ├── Car_Drive.c
    ├── tim.c
    └── main.c
```

其中：

- `Car_Drive.c`：电机运动控制
- `Car_Drive.h`：函数声明
- `tim.c`：TIM4 PWM 配置
- `main.c`：PWM 启动及功能调用

## 注意

STM32 GPIO 不能直接驱动直流电机，PB6~PB9 应连接到电机驱动器/H 桥的逻辑输入端。

`Car_Brake()` 的具体制动效果取决于实际使用的电机驱动芯片，使用前建议确认驱动芯片真值表。

## License

Original code and documentation in this repository are released under the [MIT License](./LICENSE).

STM32Cube/HAL, CMSIS, startup code, generated vendor files, and other third-party components retain their original licenses. See [NOTICE.md](./NOTICE.md) for details.


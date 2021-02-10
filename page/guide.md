# CornerStone 接口

## 硬件适配

### 电机
1. 在`handle.h`中声明`Motor_Type`类型的变量，用作储存电机数据，例：

```C
__HANDLE_EXT Motor_type Motor_LF;
```

同时检查是否有用于安全模式的全局变量：

```c
__HANDLE_EXT uint8_t Safemode;
```

2. 在`handle.h`中调用`Motor_Init()`来初始化电机，例:

```c
Motor_Init(&Motor_LF, CHASSIS_MOTOR_REDUCTION_RATE, DISABLE, ENABLE);
```

下面是`Motor_Init()`的参数：

| 参数名        | 描述                          |
| ------------- | ----------------------------- |
| Motor_Type    | 电机结构体                    |
| reductionRate | 电机减速比(输入转速:输出转速) |
| angleEnabled  | 是否启用连续角度计算          |
| inputEnabled  | 是否启用电流输出              |

3. 在`handle.c`中的CAN设备表中定义电机及对应的电调ID。例

```c
Can1_Device[ESC_ID(0x201)] = &Motor_LF;
```

4. 在`task.c`中确认是否有`Task_Can_Send()`任务，以及`Task_Sys_Init()`任务中是否启动了`Task_Can_Send()`，没有的话可以从`user/template/tasks.c`中复制。

`Task_Can_Send()`任务负责按之前定义的CAN设备表自动发送CAN数据，并在安全模式中（`SafetyMode=1`）禁用所有输出。

5. 在任务中，通过结构体成员`input`对电调进行输入，例：

```c
Motor_LF.input = 1234;
```


### Snail电机
1. 在`handle.h`中声明`PWM_Type`类型的变量
2. 在`bsp.c`中对PWM输出初始化，例如：
```C
BSP_PWM_Set_Port(&PWM_Snail1, PWM_PORT_PD12);
BSP_PWM_Init(&PWM_Snail1, 180, 1250, TIM_OCPolarity_Low);
```

### 舵机
1. 在`handle.h`中声明`PWM_Type`类型的变量
2. 在`bsp.c`中对PWM输出初始化

### 遥控器（键鼠）
1. 在`handle.h`中声明`Remote_Type`、`Keyboard_Type`和`Mouse_Type`类型的变量
2. 在`handle.c`中调用`DBUS_Init()`
3. 在`bsp.c`中调用`BSP_DBUS_Init()`
4. 在`interrupt.c`中的`USART1_IRQHandler()`函数中调用`DBus_Update()`

### 陀螺仪
1. 在`handle.h`中声明`GyroscopeData_Type`类型的变量
2. 在`Task_Sys_Init`中调用`Gyroscope_Init()`
3. 在`interrupt.c`中的`EXTI9_5_IRQHandler()`函数中调用`Gyroscope_Update()`

### 继电器
1. 在`bsp.c`中使能IO口
2. 在`handle.h`中声明宏定义，例如：  
```C
#define GO_ON GPIO_SetBits(GPIOA, GPIO_Pin_1)
#define GO_OFF GPIO_ResetBits(GPIOA, GPIO_Pin_1)
```
根据继电器上的接线以及设置，SetBits和ResetBits控制的效果不一样

### 气动电磁阀
连接继电器后通过继电器控制高低电频来控制开关

### 光电传感器
在`bsp.c`中使能IO口

### 距离传感器
1. 在`bsp.c`中声明所用IO口及计时器
2. 在`handle.h`中声明`u32`的捕获值
3. 在`interrupt.c`中设置定时器中断来捕获值，例如：  
```C
void TIM5_IRQHandler(void) {
    // 单通道输入捕获
    // 捕获 1 发生捕获事件
    if (TIM_GetITStatus(TIM5, TIM_IT_CC1) != RESET) {
        // 捕获到一个下降沿
        if (TIM5CH1_CAPTURE_STA == 1) {
            TIM5CH1_CAPTURE_STA = 0;
            //获取当前的捕获值
            TIM5CH1_CAPTURE_VAL = TIM5->CCR1;
            //设置上升沿捕获
            TIM_OC1PolarityConfig(TIM5, TIM_ICPolarity_Rising);
        } else {
            TIM5CH1_CAPTURE_STA = 1;
            TIM_SetCounter(TIM5, 0);                             //计数器清空
            TIM_OC1PolarityConfig(TIM5, TIM_ICPolarity_Falling); //设置下降沿捕获 
       }  
    }  
    TIM_ClearITPendingBit(TIM5, TIM_IT_CC1 | TIM_IT_Update); //清除中断标志位
}
```
4. 该捕获值即为距离

### PID控制器
1. 在`handle.h`中声明`PID_Type`类型的变量
1. 在任务循环前调用`PID_Init()`

### 通讯协议
1. 在`handle.h`中声明`Protocol_Type`类型的变量
2. 在`handle.c`中调用`Protocol_Init()`
3. 在`bsp.c`中调用对USART和DMA初始化，例如：  
```C
BSP_USART6_Init(115200, USART_IT_IDLE);
BSP_DMA_USART6_RX_Init(Judge.receiveBuf, Protocol_Buffer_Length);
BSP_DMA_USART6_TX_Init(Judge.sendBuf, Protocol_Buffer_Length);
```

# 第一、二次作业：LED 闪烁与按键控制

实现过程：
第一次作业：使用STM32F407VGT板 在完成了基础配置， 并编写代码使小灯闪烁
第二次作业：使用STM32F407IGH板，重新配置，在主程序中根据更改红灯闪烁时间，
课上：正常完成红灯每秒闪烁，绿灯按下后反转
课下：让定时器中断控制红灯，红灯固定由 1 秒定时器控制；按键只控制绿灯翻转

1.启动定时器代码
```c
MX_TIM3_Init();

__HAL_TIM_CLEAR_FLAG(&htim3, TIM_FLAG_UPDATE);
HAL_TIM_Base_Start_IT(&htim3);
```

2.让红灯反转
```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if (htim->Instance == TIM3)
    {
        HAL_GPIO_TogglePin(GPIOH, GPIO_PIN_12);
    }
}
```

3.按键独立控制绿灯
```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    if (GPIO_Pin == GPIO_PIN_0)
    {
        HAL_GPIO_TogglePin(GPIOH, GPIO_PIN_11);
    }
}
```

CubeMX 配置截图
第一二次作业截图
![GPIO 配置](assets/GPIO.png)
![时钟配置](assets/clock-config.png)

补充：
在 homework 文件夹的原有版本中包含 AI 辅助添加的按键消抖：
按键中断控制绿灯时，加入简单消抖。
按下按钮后，EXTI 检测上升沿，调用按键回调。50 毫秒内的重复触发会被忽略，减少机械接触抖动造成的连续翻转。中断内没有使用可能导致卡住的延时函数

## 第三次作业：UART 通信（2026-09-26）

工程目录：[two_works/03_uart/UART](two_works/03_uart/UART)。

### 硬件与配置

- 开发板：STM32F407IGH6（C 板）。外壳 UART1 接口对应芯片 USART6，代码句柄为 `huart6`。
- USART6：115200 baud、8 位数据、无校验、1 位停止位，无硬件流控。
- TX：PG14；RX：PG9。串口适配器 TX 接板上 RX，适配器 RX 接板上 TX，并保持共地。
- PH12 为红灯，PH11 为绿灯，均为推挽输出。
- USART6 全局中断已启用，`USART6_IRQHandler()` 调用 `HAL_UART_IRQHandler(&huart6)`。
- 当前工程使用 HSI 经 PLL 产生系统时钟。

### 阶段一：轮询接收

轮询版本保存在 commit `c78e03a`。通过 `HAL_UART_Receive()` 逐字节接收，随后回传原字节，并解析 `R0/R1/G0/G1` 控制红绿灯。

以下是仓库现有的阶段截图，不能代替本次中断版本的重新烧录验证：

![轮询阶段截图](assets/polling.png)
![串口通信截图](assets/communication.png)

### 阶段二：中断接收（当前代码）

1. 初始化结束后调用 `HAL_UART_Receive_IT(&huart6, &rx_byte, 1)`，启动一次单字节接收。
2. HAL 收满一个字节后调用 `HAL_UART_RxCpltCallback()`。
3. 回调只把字节放入环形队列，并重新启动下一次中断接收，不在中断内发送数据或延时。
4. 主循环从队列取出字节，使用 `HAL_UART_Transmit()` 回传，再调用 `ProcessCommand()` 控制小灯。
5. 解析器保存上一个 `R` 或 `G`，因此命令的两个字符可以分开到达；数字字符 `0`、`1` 分别表示关闭、打开。

本阶段采用中断接收、阻塞发送。队列有 256 个位置，可缓存 255 字节，面向串口助手发送短命令的实验。队列满、UART 出错或重新启动接收失败时设置 `rx_fault`，主循环进入 `Error_Handler()`；当前未实现错误自动恢复。

保留原工程的 `LED_ON = GPIO_PIN_SET`、`LED_OFF = GPIO_PIN_RESET`；实际亮灭极性需要在板上确认，若反向则交换这两个宏。

### 编译与待验证项目

- 已执行 `cmake --build --preset Debug`，编译、链接成功，无编译器警告。
- 当前构建 FLASH 使用 11424 B，RAM 使用 1920 B。
- 尚未由本次修改进行烧录或实物测试，以下结果待实测填写。

串口助手设置为 115200、8N1、文本发送，关闭本地回显。烧录中断版本后依次测试：

| 输入 | 预期结果 | 本次实测 |
| --- | --- | --- |
| `Hello` | 原样回传 `Hello` | 待验证 |
| `R1` / `R0` | 红灯亮 / 灭，同时回传命令 | 待验证 |
| `G1` / `G0` | 绿灯亮 / 灭，同时回传命令 | 待验证 |
| `R1G0` | 红灯亮、绿灯灭，同时回传命令 | 待验证 |
| 先发送 `R`，再发送 `1` | 红灯亮，两个字符分别回传 | 待验证 |

测试完成后填写真实现象，并另存中断阶段截图。当前仅完成到中断接收阶段，作业要求的 DMA 收发仍需后续实现和验证。


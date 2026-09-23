# 两个独立工程：原有 Homework 与 EXTI/TIM

本目录保存 STM32F407IGH6 C 板的两个完整工程，各自包含源码、CubeMX 配置、驱动和构建文件。

| 目录 | 内容 |
| --- | --- |
| [01_homework](01_homework) | 仓库原有 `stm32f407igh6` 作业工程的完整副本，源码保持一致 |
| [02_exti_tim](02_exti_tim) | TIM3 每秒翻转红灯，按键独立翻转绿灯，不加消抖 |

## 两份代码的区别

`01_homework` 来自仓库原有的 `stm32f407igh6`，保持其代码和配置：TIM3 每秒翻转红灯，PA0 按键切换绿灯，带原有的 50 ms 时间窗口消抖。它不是 Clocksystemblack 修改前的快照。

`02_exti_tim` 来自 Clocksystemblack：移除原来的 `state` 变速逻辑和主循环延时闪灯，改由 TIM3 中断每秒翻转 PH12 红灯；PA0 按键中断只翻转 PH11 绿灯，不改变红灯节奏，不添加消抖。

第二个版本的执行流程：

1. TIM3 初始化及其必要的时钟、NVIC、中断入口和 HAL 驱动编译配置。
2. 在 `USER CODE BEGIN 2` 中调用 `HAL_TIM_Base_Start_IT(&htim3)`。
3. 在 `USER CODE BEGIN 4` 中实现 `HAL_TIM_PeriodElapsedCallback`，每次 TIM3 更新中断翻转 PH12 红灯。
4. `HAL_GPIO_EXTI_Callback` 收到 PA0 中断时翻转 PH11 绿灯。主循环保持空闲，不调用阻塞延时。

TIM3 的现有配置是 PSC=16799、ARR=4999，时钟为 84 MHz，因此每秒产生一次更新中断。启动前清除初始化产生的更新标志，使第一次翻转在约一秒后发生。红灯亮一秒、灭一秒，完整亮灭周期为两秒；原来的 `tim3_count` 已移除。

本版本按用户最新要求实现定时红灯和独立按键绿灯，不加入消抖。真实按键的机械抖动可能导致一次按压出现多次绿灯翻转，这是未消抖版本的实际限制。

## 打开与编译

在 VS Code 中用“打开文件夹”单独打开 `01_homework` 或 `02_exti_tim`。其下载配置使用当前文件夹中的固件，不会跳回原始工程目录。

在所选工程的终端中执行：

```powershell
cmake --preset Debug
cmake --build --preset Debug
```

需要 PATH 中有 CMake、Ninja 和 Arm GNU Toolchain。烧录、调试还需要 OpenOCD、匹配的下载器以及 Cortex-Debug 扩展。`01_homework` 的固件为 `build/Debug/board.elf`，`02_exti_tim` 的固件为 `build/Debug/Clocksystemblack.elf`。

## 观察运行效果

选择 `02_exti_tim` 的 DAP 或 STLink 调试配置，启动调试后继续运行。红灯应亮一秒、灭一秒，按键应翻转绿灯且不改变红灯节奏；两盏灯保持原配置，启动时均为高电平点亮。实际结果需连接开发板验证。

两个工程用途不同：`01_homework` 保留原有作业及其消抖逻辑；`02_exti_tim` 是按最新要求实现的定时红灯、独立按键绿灯练习，不加消抖。

两个目录是独立副本。本次在原始 Clocksystemblack 中修改后同步覆盖 `02_exti_tim`；原有作业 `stm32f407igh6` 和 `01_homework` 保持原样。

作业截图和个人说明仍由作者填写在仓库根目录的 `LOG.md`，本目录不代写实测结果。

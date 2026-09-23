# 两个独立工程：EXTI 与 TIM3

本目录保存 STM32F407IGH6 C 板的两个完整工程，各自包含源码、CubeMX 配置、驱动和构建文件。

| 目录 | 内容 |
| --- | --- |
| [01_exti](01_exti) | Clocksystemblack 修改前的原按键版本 |
| [02_exti_tim](02_exti_tim) | 在原代码基础上，仅增加 TIM3 中断和每秒计数 |

## 两份代码的区别

两个版本都保留原来的 `int state = 1`、主循环闪灯逻辑、`HAL_Delay(2000)` / `HAL_Delay(1000)`，以及 PA0 中断中切换 `state` 和翻转 PH11 绿灯的逻辑。

第二个版本只新增：

1. TIM3 初始化及其必要的时钟、NVIC、中断入口和 HAL 驱动编译配置。
2. 在 `USER CODE BEGIN 2` 中调用 `HAL_TIM_Base_Start_IT(&htim3)`。
3. 在 `USER CODE BEGIN 4` 中实现 `HAL_TIM_PeriodElapsedCallback`，每次 TIM3 更新中断将 `tim3_count` 加一。

`tim3_count` 声明为 `volatile uint32_t`，用于观察中断次数。TIM3 的现有配置是 PSC=16799、ARR=4999，时钟为 84 MHz，因此每秒产生一次更新中断。启动前清除初始化产生的更新标志，使第一次计数在约一秒后发生。

截图里的 TODO 未指定动作；本版本按用户选择实现计数，不在定时器回调中修改任何灯的电平，也没有额外添加按键消抖或改写原控制逻辑。

## 打开与编译

在 VS Code 中用“打开文件夹”单独打开 `01_exti` 或 `02_exti_tim`。其下载配置使用当前文件夹中的固件，不会跳回原来的 Clocksystemblack 目录。

在所选工程的终端中执行：

```powershell
cmake --preset Debug
cmake --build --preset Debug
```

需要 PATH 中有 CMake、Ninja 和 Arm GNU Toolchain。烧录、调试还需要 OpenOCD、匹配的下载器以及 Cortex-Debug 扩展。每份工程的固件各自位于 `build/Debug/Clocksystemblack.elf`。

## 观察定时器计数

选择 `02_exti_tim` 的 DAP 或 STLink 调试配置，启动调试后继续运行。可以使用已有的 Live Watch 查看 `tim3_count`；若当前调试器不支持运行中读取，则让程序运行数秒后暂停，在 Watch 中查看该变量。预计约每运行一秒增加一次，实际结果需连接开发板验证。

两份工程的红灯与按键行为应一致。计数器是新增 TIM3 中断生效的观察点，不是新的闪灯控制器。

`01_exti` 保留原工程，其中 `.ioc` 已有 TIM3 参数但原 C 代码尚未启用 TIM3。这是原状态快照；若重新用 CubeMX 生成代码，它可能改变快照。查看已有配置可以打开 `.ioc`，无需重新生成。

作业截图和个人说明仍由作者填写在仓库根目录的 `LOG.md`，本目录不代写实测结果。

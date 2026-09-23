# STM32F407 C 板作业工程

本仓库使用 STM32F407IGH6（RoboMaster C 型开发板），基于 [PNX 作业模板](https://github.com/HKUSTGZ-ROBOMASTER-PNX/stm32f407_hw_template)。

当前工程来自已完成的 Clocksystemblack，包含系统时钟、GPIO、按键 EXTI 和 TIM3 定时中断。仓库只保留实际使用的 C 板工程，作业截图与个人说明由作者在 `LOG.md` 中补充。

## 工程位置

| 文件或目录 | 用途 |
| --- | --- |
| `stm32f407igh6/board.ioc` | CubeMX 配置，使用 CubeMX 6.18.1 和 STM32Cube F4 V1.28.3 |
| `stm32f407igh6/Core/Src/main.c` | 主程序、GPIO/TIM3 初始化和中断回调 |
| `stm32f407igh6/Core/Src/stm32f4xx_it.c` | EXTI0、TIM3 中断入口 |
| `stm32f407igh6/Drivers` | 工程所需的 CMSIS 和 HAL 驱动 |
| `.vscode` | 从仓库根目录打开时使用的构建、烧录和调试配置 |
| `LOG.md` | 个人实验记录，待作者填写 |
| `assets` | 图片资料，可放入真实实验截图 |

## 当前功能

- PH12 红灯由 TIM3 中断每 1 秒翻转一次，即亮 1 秒、灭 1 秒，完整周期 2 秒。
- PA0 配置为下拉、上升沿 EXTI0；有效按键事件翻转 PH11 绿灯。
- 按键采用 50 ms 简单时间窗口消抖，中断中不调用 `HAL_Delay`。
- 保留原工程初始高电平，红绿灯启动时均亮。
- 按键不改变红灯节奏；原来的 `state` 变速逻辑已由固定周期定时器替代。

时钟设置为 HSE 8 MHz、SYSCLK 168 MHz、APB1 42 MHz、TIM3 84 MHz。TIM3 的 PSC 为 16799、ARR 为 4999：

```text
更新中断间隔 = (16799 + 1) × (4999 + 1) / 84000000 = 1 秒
```

## 构建与运行

需要 CMake 3.22 或更新版本、Ninja、Arm GNU Toolchain。烧录还需要 OpenOCD 和实际下载器；VS Code 调试需要 Cortex-Debug 扩展。上述可执行程序应在 PATH 中。

在仓库根目录打开 VS Code，按 `Ctrl+Shift+B` 构建；或在 PowerShell 中执行：

```powershell
Set-Location -LiteralPath './stm32f407igh6'
cmake --preset Debug
cmake --build --preset Debug
```

固件生成在 `stm32f407igh6/build/Debug/board.elf`。在 VS Code 中选择匹配的 STLink 或 DAP 调试配置，按 F5 启动调试；停在 `main` 后继续运行。也可以通过“运行任务”选择 `flash-stlink` 或 `flash-dap` 烧录。

确认红灯每秒翻转，按键可以切换绿灯且不影响红灯节奏。代码构建与链接检查不能代替真实开发板验收，运行现象应由作者在实测后记录。

## 版本管理

`origin` 应指向个人仓库，`upstream` 指向课程模板仓库。构建产物已通过 `.gitignore` 排除。

提交说明使用 `feat: ...`、`fix: ...`、`chore: ...` 等格式。提交前查看 `git status` 和 `git diff`，将真实截图和个人记录与对应代码一起纳入版本管理。

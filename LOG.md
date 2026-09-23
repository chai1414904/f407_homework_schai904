第一·二次作业：[让 LED 灯闪烁并完成红灯每秒闪烁，绿灯按下后反转]

实现过程
[第一次作业：使用STM32F407VGT板 在完成了基础配置， 并编写代码使小灯闪烁
第二次作业：使用STM32F407IGH板，重新配置，在主程序中根据更改红灯闪烁时间，
课上：正常完成红灯每秒闪烁，绿灯按下后反转
课下：让定时器中断控制红灯，红灯固定由 1 秒定时器控制；按键只控制绿灯翻转]

1.启动定时器代码
MX_TIM3_Init();

__HAL_TIM_CLEAR_FLAG(&htim3, TIM_FLAG_UPDATE);
HAL_TIM_Base_Start_IT(&htim3);

2.让红灯反转
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if (htim->Instance == TIM3)
    {
        HAL_GPIO_TogglePin(GPIOH, GPIO_PIN_12);
    }
}
3.按键独立控制绿灯
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    if (GPIO_Pin == GPIO_PIN_0)
    {
        HAL_GPIO_TogglePin(GPIOH, GPIO_PIN_11);
    }
}

CubeMX 配置截图
![第一，二次作业配置]
(assets/GPIO.png)(assets/clock--config.png)

补充：
[在homework 文件夹里有AI的消抖：
按键中断控制绿灯时，加入简单消抖。
按下按钮后，EXTI 检测上升沿，调用按键回调。50 毫秒内的重复触发会被忽略，减少机械接触抖动造成的连续翻转。中断内没有使用可能导致卡住的延时函数



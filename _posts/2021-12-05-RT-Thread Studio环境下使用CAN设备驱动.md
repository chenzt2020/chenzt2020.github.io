---
title: RT-Thread Studio环境下使用CAN设备驱动
tags:
  - RT-Thread
  - CAN
  - STM32
---

本文以 STM32F407ZG 为例

### 1 双击 RT-Thread Settings -> 更多配置 -> 使用CAN设备驱动程序 -> Ctrl+S

### 2 在 stm32f4xx_hal_conf.h 文件中打开 HAL_CAN_MODULE_ENABLED 宏定义

### 3 在 rtconfig.h 文件里添加 RT_USING_CAN1 BSP_USING_CAN BSP_USING_CAN1 的宏定义
``` cpp
#define RT_USING_CAN
#define RT_USING_CAN1
#define BSP_USING_CAN
#define BSP_USING_CAN1
```

### 4 在 board.c 中添加函数

``` cpp
void HAL_CAN_MspInit(CAN_HandleTypeDef* hcan)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};
  if(hcan->Instance==CAN1)
  {
  /* USER CODE BEGIN CAN1_MspInit 0 */

  /* USER CODE END CAN1_MspInit 0 */
    /* Peripheral clock enable */
    __HAL_RCC_CAN1_CLK_ENABLE();

    __HAL_RCC_GPIOA_CLK_ENABLE();
    /**CAN1 GPIO Configuration
    PA11     ------> CAN1_RX
    PA12     ------> CAN1_TX
    */
    GPIO_InitStruct.Pin = GPIO_PIN_11|GPIO_PIN_12;
    GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_VERY_HIGH;
    GPIO_InitStruct.Alternate = GPIO_AF9_CAN1;
    HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  /* USER CODE BEGIN CAN1_MspInit 1 */

  /* USER CODE END CAN1_MspInit 1 */
  }

}
```

### 5 将主程序改为[例程](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/can/can?id=can-%e8%ae%be%e5%a4%87%e4%bd%bf%e7%94%a8%e7%a4%ba%e4%be%8b)

### 6 添加 drv_can.c drv_can.h 并修改 drv_can.c
将
```cpp
#elif defined (SOC_SERIES_STM32F4)/* APB1 45MHz(max) */
static const struct stm32_baud_rate_tab can_baud_rate_tab[] =
{
    {CAN1MBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_5TQ | 3)},
    {CAN800kBaud, (CAN_SJW_2TQ | CAN_BS1_8TQ  | CAN_BS2_5TQ | 4)},
    {CAN500kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_5TQ | 6)},
    {CAN250kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_5TQ | 12)},
    {CAN125kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_5TQ | 24)},
    {CAN100kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_5TQ | 30)},
    {CAN50kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_5TQ | 60)},
    {CAN20kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_5TQ | 150)},
    {CAN10kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_5TQ | 300)}
};
```
改为
```cpp
#elif defined (SOC_SERIES_STM32F4)/* APB1 42MHz(max) */
static const struct stm32_baud_rate_tab can_baud_rate_tab[] =
{
    {CAN1MBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_4TQ | 3)},
    {CAN800kBaud, (CAN_SJW_2TQ | CAN_BS1_8TQ  | CAN_BS2_4TQ | 4)},
    {CAN500kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_4TQ | 6)},
    {CAN250kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_4TQ | 12)},
    {CAN125kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_4TQ | 24)},
    {CAN100kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_4TQ | 30)},
    {CAN50kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_4TQ | 60)},
    {CAN20kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_4TQ | 150)},
    {CAN10kBaud, (CAN_SJW_2TQ | CAN_BS1_9TQ  | CAN_BS2_4TQ | 300)}
};
```
---
title: RT-Thread+STM32+CanFestival
tags:
  - RT-Thread
  - CanFestival
  - CanOpen
  - CAN
  - STM32
---

本文以STM32F407ZG为例

### 1 新建 RT-Thread 项目

基于芯片创建 RT-Thread 项目，芯片选 STM32F407ZG，然后按照[该文](/_posts/2021-12-05-RT-Thread-Studio环境下使用CAN设备驱动/)中的方法配置 CAN 设备，第5步是为了测试用，可以跳过

此时 RT-Thread 项目已经可以使用 CAN 设备进行收发，接下来我们要把 CanFestival 添加进项目

### 2 将 CanFestival 代码添加进项目

按照[该文](/_posts/2021-12-05-CanFestival移植到STM32/)中的方法整理 CanFestival 的代码，然后将 CanFestival 目录复制到 RT-Thread 项目的 libraries 文件夹下，然后进行如下操作：
```
右键项目 -> 属性 -> C/C++ 常规 -> 路径和符号 -> 添加 //${ProjName}/libraries/CanFestival/include 和 //${ProjName}/libraries/CanFestival/include/STM32，注意勾选“是一个工作空间路径” -> 应用并关闭 -> 是 -> 重新构建
```
此时已经将 CanFestival 代码添加进项目，但尚未实现接口函数

### 3 用 RT-Thread 的 CAN 设备操作实现 CanFestival 的接口

例程如下，参考 RT-Thread 官方文档中的 [CAN 设备使用示例](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/can/can?id=can-%e8%ae%be%e5%a4%87%e4%bd%bf%e7%94%a8%e7%a4%ba%e4%be%8b)

main.c
```cpp
#include <rtthread.h>

#define DBG_TAG "main"
#define DBG_LVL DBG_LOG
#include <rtdbg.h>

extern int can_init();
int main(void)
{
    can_init();
    return RT_EOK;
}
```

my_can.c
```cpp
/*
 * Copyright (c) 2006-2021, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2021-12-02     czt       the first version
 */

#include <rtthread.h>
#include "rtdevice.h"
#include "canfestival.h"
#include "TestMaster.h"

#define CAN_DEV_NAME "can1" /* CAN 设备名称 */
#define CANOPEN_DEBUG

static struct rt_semaphore rx_sem; /* 用于接收消息的信号量 */
static rt_device_t can_dev; /* CAN 设备句柄 */

static rt_err_t can_rx_call(rt_device_t dev, rt_size_t size)
{
    /* CAN 接收到数据后产生中断，调用此回调函数，然后发送接收信号量 */
    rt_sem_release(&rx_sem);
    return RT_EOK;
}

static void can_rx_thread(void *parameter)
{
    /* 设置接收回调函数 */
    rt_device_set_rx_indicate(can_dev, can_rx_call);

#ifdef RT_CAN_USING_HDR
    rt_err_t res;
    struct rt_can_filter_item items[5] =
    {
        RT_CAN_FILTER_ITEM_INIT(0x100, 0, 0, 1, 0x700, RT_NULL, RT_NULL), /* std,match ID:0x100~0x1ff，hdr 为 - 1，设置默认过滤表 */
        RT_CAN_FILTER_ITEM_INIT(0x300, 0, 0, 1, 0x700, RT_NULL, RT_NULL), /* std,match ID:0x300~0x3ff，hdr 为 - 1 */
        RT_CAN_FILTER_ITEM_INIT(0x211, 0, 0, 1, 0x7ff, RT_NULL, RT_NULL), /* std,match ID:0x211，hdr 为 - 1 */
        RT_CAN_FILTER_STD_INIT(0x486, RT_NULL, RT_NULL), /* std,match ID:0x486，hdr 为 - 1 */
        {   0x555, 0, 0, 1, 0x7ff, 7,} /* std,match ID:0x555，hdr 为 7，指定设置 7 号过滤表 */
    };
    struct rt_can_filter_config cfg =
    {   5, 1, items}; /* 一共有 5 个过滤表 */
    /* 设置硬件过滤表 */
    res = rt_device_control(can_dev, RT_CAN_CMD_SET_FILTER, &cfg);
    RT_ASSERT(res == RT_EOK);
#endif

    struct rt_can_msg msg = { 0 };
    /* CanFestival初始化 */
    setNodeId(&TestMaster_Data, 0x21);
    setState(&TestMaster_Data, Initialisation);

    while (1)
    {
        /* hdr 值为 - 1，表示直接从 uselist 链表读取数据 */
        msg.hdr = -1;
        /* 阻塞等待接收信号量 */
        rt_sem_take(&rx_sem, RT_WAITING_FOREVER);
        rt_device_read(can_dev, 0, &msg, sizeof(msg));

        Message m = Message_Initializer;
        m.cob_id = msg.id;
        m.rtr = msg.rtr;
        m.len = msg.len;
        memcpy(m.data, msg.data, sizeof(UNS8) * msg.len);

        canDispatch(&TestMaster_Data, &m);
#ifdef CANOPEN_DEBUG
        rt_kprintf("can rx: %xh ", msg.id);
        for (int i = 0; i < 8; i++)
        {
            rt_kprintf("%3x", msg.data[i]);
        }
        rt_kprintf("\n");
#endif
    }
}

int can_init(int argc, char *argv[])
{
    rt_err_t res;
    rt_thread_t thread;

    /* 查找 CAN 设备 */
    can_dev = rt_device_find(CAN_DEV_NAME);
    if (!can_dev)
    {
        rt_kprintf("find %s failed!\n", CAN_DEV_NAME);
        return RT_ERROR;
    }

    /* 初始化 CAN 接收信号量 */
    rt_sem_init(&rx_sem, "rx_sem", 0, RT_IPC_FLAG_FIFO);

    /* 以中断接收及发送方式打开 CAN 设备 */
    res = rt_device_open(can_dev, RT_DEVICE_FLAG_INT_TX | RT_DEVICE_FLAG_INT_RX);
    RT_ASSERT(res == RT_EOK);
    /* 创建数据接收线程 */
    thread = rt_thread_create("can_rx", can_rx_thread, RT_NULL, 1024, 25, 10);
    if (thread != RT_NULL)
    {
        rt_thread_startup(thread);
    }
    else
    {
        rt_kprintf("create can_rx thread failed!\n");
    }
    return res;
}

/* CanOpen发送数据接口 */
UNS8 canSend(CAN_PORT notused, Message *m)
{
    struct rt_can_msg msg = { 0 };
    rt_size_t size;

    msg.id = m->cob_id;
    msg.ide = RT_CAN_STDID;
    msg.rtr = m->rtr;
    msg.len = m->len;
    memcpy(msg.data, m->data, sizeof(UNS8) * m->len);

    /* 发送 CAN 数据 */
    size = rt_device_write(can_dev, 0, &msg, sizeof(msg));
#ifdef CANOPEN_DEBUG
    rt_kprintf("can tx: %xh ", msg.id);
    for (int i = 0; i < 8; i++)
    {
        rt_kprintf("%3x", msg.data[i]);
    }
    rt_kprintf("\n");
    if (size == 0)
    {
        rt_kprintf("can dev write data failed!\n");
    }
#endif
    return size == 0;
}

TIMEVAL getElapsedTime(void)
{
    return 0;
}
void setTimer(TIMEVAL value)
{

}
```
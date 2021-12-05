---
title: CanFestival移植到STM32
tags:
  - CanFestival
  - CanOpen
  - STM32
  - CAN
---

### 1 获取 CanFestival 的源代码

[CanFestival 官网](https://canfestival.org/code.html.en)

### 2 整理源代码

将源代码和 AVR 架构的头文件提取出来，然后删除和 AVR iar 相关的头文件，即删除 can_AVR.h can_drv.h iar.h，最后整理成如下结构
```
CanFestival
├── include
│   ├── can_driver.h
│   ├── can.h
│   ├── data.h
│   ├── dcf.h
│   ├── def.h
│   ├── emcy.h
│   ├── lifegrd.h
│   ├── lss.h
│   ├── nmtMaster.h
│   ├── nmtSlave.h
│   ├── objacces.h
│   ├── objdictdef.h
│   ├── pdo.h
│   ├── sdo.h
│   ├── states.h
│   ├── STM32
│   │   ├── applicfg.h
│   │   ├── canfestival.h
│   │   ├── config.h
│   │   └── timerscfg.h
│   ├── sync.h
│   ├── sysdep.h
│   ├── timers_driver.h
│   └── timers.h
└── src
    ├── dcf.c
    ├── emcy.c
    ├── lifegrd.c
    ├── lss.c
    ├── nmtMaster.c
    ├── nmtSlave.c
    ├── objacces.c
    ├── pdo.c
    ├── sdo.c
    ├── states.c
    ├── sync.c
    └── timer.c
```

### 3 修改 config.h
将 config.h 文件中关于 iar 的头文件引用删除，即删除以下行
``` cpp
#ifdef  __IAR_SYSTEMS_ICC__
#include <ioavr.h>
#include <intrinsics.h>
#include "iar.h"
#else	// GCC
#include <inttypes.h>
#include <avr/io.h>
#include <avr/interrupt.h>
#include <avr/pgmspace.h>
#include <avr/sleep.h>
#include <avr/wdt.h>
#endif	// GCC
```

### 4 实现3个接口函数
``` cpp
UNS8 canSend(CAN_PORT notused, Message *m)
{
    return 0;
}
TIMEVAL getElapsedTime(void)
{
    return 0;
}
void setTimer(TIMEVAL value)
{

}
```

### 5 添加对象字典
建立自己的对象字典，并将源文件与头文件分别放入 src 文件夹和 include 文件夹中，对象字典需要用对象字典编辑器编辑
```
.
├── include
│   └── TestMaster.h
└── src
    └── TestMaster.c
```

### 6 初始化 CanOpen
在主文件中引用必要的头文件
```cpp
#include "canfestival.h"
#include "TestMaster.h"
```
并对 CanOpen 进行初始化
```cpp
setNodeId(&TestMaster_Data, 0x21);
setState(&TestMaster_Data, Initialisation);
```
此时即可进行 CanOpen 的使用
```cpp
canDispatch(&TestMaster_Data, &m);
```
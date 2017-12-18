---
title: "啟用 GPIO 的特殊功能"
date: 2016-01-02T04:14:13+08:00
categories:
  - "Embedded"
tags:
  - "C/C++"
---
MCU 的 GPIO 上面會有各式各樣的特殊功能在裡頭，像是 I2C、UART、PWM 等等，不過，要使用這些功能以前，需要先對這些 GPIO 做微調，否則腳位們依舊只會表現為平常的 GPIO。
<!--more-->

這一陣子在使用 TI 的 TM4C123 系列的 MCU（它長在 Tiva Launchpad 上面），誠如開發版的名字，使用的函式庫叫做 TivaWare。這函式庫的使用上有利有弊其實，個人沒有很喜歡它，但先撇開這部分不提。
正在寫一個 I2C 的 wrapper，可以一個介面操控 MCU 上面的 4 個 I2C 的 I/O。不過測試時，除了第一組 I2C 模組會動，其他的一概不會傳出信號。稍微除了錯以後發現沒有把 GPIO 的多工器切換到 I2C，因此 I2C 模組的信號沒辦法輸出到外部 :p

下面以 I2C 為例：
```c
SysCtlPeripheralEnable(SYSCTL_PERIPH_I2C0);

GPIOPinConfigure(GPIO_PB2_I2C0SCL);
GPIOPinTypeI2CSCL(GPIO_PORTB_BASE, GPIO_PIN_2);

GPIOPinConfigure(GPIO_PB3_I2C0SDA);
GPIOPinTypeI2CSDA(GPIO_PORTB_BASE, GPIO_PIN_3);
```

這段程式經過了三個大步驟：
1. `SysCtlPeripheralEnable` 把模組的時脈來源打開。
2. `GPIOPinConfigure` 把該腳位前端的多工器切換到想使用的模組，以第三行為例，就是把 PB2 的多工器，切換到 I2C 模組編號 0 的 SCL。
3. `GPIOPinTypeI2CSCL` 對這個腳位做細部的 I/O 控制，像是 pull-up resistor 要不要使用、該腳位是輸入還是輸出⋯⋯等。

而測試的時候，只有在初始化的時候有使用到 `GPIOPinConfigure` 來切換多工器，後面的初始化（也就是除了 I2C0 以外的模組，I2C1 到 I2C3），全部都沒有做這的動作。因此 I2C 的初始雖然完成了，但信號不會出來。

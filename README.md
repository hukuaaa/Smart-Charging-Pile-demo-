# 基于 STM32 + RT-Thread 的智能汽车充电桩系统（简化版）

## 🚀 项目效果演示
<p align="center">
  <img src="https://github.com/user-attachments/assets/1b9d3aa4-5b33-4ce0-bf26-4556c95f947c" alt="项目核心功能演示" width="300">
</p>

## 💡 项目简介
本项目是一款基于 **STM32F103ZET6** 主控芯片与 **RT-Thread (RTT) 实时操作系统** 开发的智能汽车充电桩系统。

针对传统单片机裸机开发在处理网络通信时容易造成的“系统阻塞、实时性差”等痛点，引入了 RT-Thread 操作系统进行多线程并发管理。项目成功打通了**“底层感知（RFID刷卡/LCD交互） -> 传输层（ESP8266 Wi-Fi） -> 平台层（巴法云MQTT） -> 应用层（微信小程序）”**的智能物联网全链路闭环。

---

## 🛠️ 核心功能特性
* **多线程并发控制**：利用 RT-Thread 内核，实现 LED 提示、按键捕获、UI 刷新、Wi-Fi 通信 4 大核心线程并发协作，分时复用 CPU，确保系统具备高实时性与零卡顿的人机交互体验。
* **射频识别（RFID）系统**：基于 SPI 协议驱动 MFRC522 模块。利用非接触式电磁感应原理自动唤醒 IC 卡芯片，实现高可靠性的寻卡、校验、防冲突算法及刷卡充扣费业务。
* **高帧率本地人机交互**：通过 2.8 英寸 TFT-LCD 彩色液晶屏（RGB565 颜色格式），实现充电状态、当前电量、消费金额等关键数据的毫秒级动态 UI 刷新。
* **端云双向同步通信**：通过串口（UART）驱动 ESP8266 模块，使用 AT 指令与巴法云（Bemfa Cloud）MQTT 服务器建立长连接，实现本地数据的“实时上行上报”与云端控制指令的“异步下行控制”。

---
<p align="center">
  <img width="1279" height="1706" alt="Image" src="https://github.com/user-attachments/assets/a325cd46-23fa-47a9-b51f-d902f5d7eb27" />
</p>


## 🏗️ 系统软件架构（多线程设计）
系统上电后，通过调用 RT-Thread 线程初始化 API 动态创建了多个任务，并在后台由 RTT 调度器进行统一优先级调度，具体业务逻辑如下：

### 1. 核心主控初始化代码展示
```c
#include <rtthread.h>
#include <rtdevice.h>
#include <board.h>
#include "led.h"
#include "beep.h"
#include "key.h"
#include "bsp_lcd.h"
#include "mfrc522.h"
#include "wifi.h"

int main(void)
{
    /* 1. 基础硬件与通信外设初始化 */
    Led_Config();         // LED指示灯初始化
    Beep_Config();        // 蜂鸣器驱动初始化
    Key_Config();         // 物理按键事件初始化
    bsp_TFTLCD_Init();    // 彩色TFT-LCD屏驱动初始化
    PCD_Init();           // RFID（MFRC522射频芯片）硬初始化并开启天线
    
    /* 2. 启动 RT-Thread 多线程并发任务 */
    WiFi_Init();          // 启动 Wi-Fi 数据上行/下行物联网通信线程
    led_thread_init();    // 启动 充电状态LED闪烁提示线程
    key_thread_init();    // 启动 按键状态死守与紧急停止线程
    view_thread_init();   // 启动 2.8寸本地LCD屏UI高帧率刷新线程

    return RT_EOK;
}


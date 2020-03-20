# ESP32Catalog

## ESP32 入门知识

本章目的：介绍 ESP32 基础知识（外设、Wi-Fi、Bluetooth等）、ESP32 芯片/模组/开发板介绍、ESP32 开发框架介绍（IDF 结构、IDF 提供功能）、ESP32 开发所涉及的知识概述（FreeRTOS、Wi-Fi、Bluetooth等）、物联网开发基础知识。

通过本章内容，让读者了解到如何学习 ESP32、如何查找学习资料、物联网应用开发概述。

1. ESP32 开发板入门（芯片、模组、开发板）
2. ESP32 软件开发 SDK 介绍（IDF、ADF、MDF）
3. ESP32 芯片介绍
4. ESP32 学习方法
5. 开发环境搭建
6. 程序下载、调试（GdbStub、Jtag Debugging）
7. 新建工程模板，工程结构

## ESP32 系统知识

本章目的：了解 ESP32 编译系统（Make/Cmake）、启动流程、日志功能、错误处理、事件处理等。

通过本章内容，让读者了解到如何学习 ESP32、如何查找学习资料、物联网应用开发概述。

1. CMake/Make 编译系统
2. Bootloader 引导流程及相关功能
3. Logging 日志功能
4. Memory layout 内存布局讲解
5. Event loop 事件注册机制
6. Inter Processor Call IPC 机制
7. Console
8. esp\_ringbuf
9. Error Handling

## ESP32 FreeRTOS

本章目的：了解 FreeRTOS 基础知识及相关概念，FreeRTOS 在 ESP32 上特殊功能。

通过本章内容，让读者了解到 FreeRTOS 基本概念，学会在 ESP32 上使用 FreeRTOS 所提供的功能。

1. FreeRTOS 介绍（ESP32 SMP）
2. FreeRTOS 配置
3. 任务创建、删除、挂起、恢复、调度等
4. 队列创建、删除
5. 信号量、二进制信号量、互斥量、计数型信号量
6. FreeRTOS 软件定时器

## ESP32 调试手段

本章目的：学习使用不同的方式进行 ESP32 调试：内存调试、Crash 调试、JTAG 调试。

1. Monitor/Backtrace
2. Heap Memory Debug
3. Core Dump
4. Gcov
5. Application Level Tracing
6. GdbStub
7. Jtag Debugging

## ESP32 存储系统

本章目的：了解 ESP32 上可利用的存储方式，详解 NVS 原理、SPIFFS/FatFS 文件系统使用、Flash 操作等

通过本章内容，让读者能根据不同的需求选择存储系统。

1. SPI Flash
2. Partition
3. NVS
4. FatFS
5. SPIFFS
6. Wear Levelling
7. VFS

## ESP32 基础外设

本章目的：ESP32 基础外设原理讲解，并针对每个外设有个对应的小示例讲解。

通过本章内容，让读者学会使用 ESP32 基础外设，并能处理常见问题，学会外设调试方法。

1. IO Matrix
2. GPIO
3. 定时器
4. 看门狗
5. UART
6. SPI
7. I2C
8. I2S
9. LEDC
10. MCPWM

## ESP32 高级外设

本章目的：ESP32 高级外设原理讲解，并针对每个外设有个对应的小示例讲解。

通过本章内容，让读者学会使用 ESP32 高级外设，并能处理常见问题，学会外设调试方法。

1. 电容触摸
2. ADC
3. DAC
4. PCNT
5. RMT
6. SDIO
7. CAN
8. Sigma-delta Modulation
9. SDMMC Host
10. SD SPI Host
11. Modbus

## Network

### ESP32 Wi-Fi

本章目的：介绍 Wi-Fi 基础知识，学会使用 Wi-Fi 接入 IP 网络并能与其他设备进行通信。

通过本章内容，让读者学会使用 Wi-Fi 基础功能（Wi-Fi 连接、配置），sniffer，ESP-NOW

1. ESP32 Wi-Fi 介绍
2. ESP32 Wi-Fi Station/AP/Sniffer
3. 网络配置（SoftAP、Smartconfig、BLufi）
4. sniffer
5. ESP-NOW
6. Provisioning

### ESP32 Protocols

本章目的：介绍网络协议并能使用与其他设备进行通信。

通过本章内容，让读者了解并学会使用网络协议。

1. TCP/IP 协议栈
2. TCP/UDP
3. TLS/SSL（mbed TLS/open SSL）
4. HTTP/HTTPS
5. HTTP server
6. SNTP
7. MDNS
8. MQTT
9. COAP
10. PPPOS

### ESP32 Bluetooth

本章目的：介绍蓝牙（BT/BLE） 基础知识、BT 相关协议、BLE 相关概念及协议。

1. ESP32 蓝牙介绍
2. BT ADV
3. BT A2DP
4. BT SPP
5. BLE ADV
6. BLE GATT

### ESP32 Ethernet

1. Ethernet
2. iperf

### ESP-MESH

1. ESP-MESH 框架介绍
2. 网络配置
3. 无路由/有路由
4. MESH 固件升级解决方案

### ESP32 低功耗

1. Power Management
2. Deep Sleep Wakeup stub
3. Sleep modes
4. ULP

## Application

本章将针对不同的应用场景进行相关指导，通过相对完整的工程学会使用 ESP32 进行完整的应用开发。

### Embedded GUI

1. 嵌入式 GUI 相关介绍
2. ESP32 上 GUI 开发过程
3. 图片、文字显示
4. 与 ESP32 外设交互

### Cloud Platform

1. 云平台介绍
2. ESP32 接入 Aliyun
3. 设备管理、量产预置等
4. 固件升级介绍

### Camera

1. OV3600 摄像头使用
2. ESP-EYE 微信小程序应用

### MESH Application

1. MESH + Cloud platform

### Others

## References


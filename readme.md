# 2023 Chongqing university Infantry MX5
# 2023赛季重庆大学 5号步兵

## 简介
2023赛季5步兵底盘采用麦克纳姆轮，配有一个上下置摩擦轮的发射机构，采用中上供弹方式。机器人配备有MiniPC，自瞄功能的实现。云台和底盘分别使用一块C板，两块板之间采用CAN通信。

![五号步兵定妆照](img\五号步兵定妆照.png)

## 基本功能
1. 小陀螺，底盘跟随云台，锁云台等基础运功能
2. 连续发射功能
3. 能够根据裁判系统获得机器人的实时数据
4. 通过底盘功率算法，保证机器人不超功率
5. 根据不同的模式（功率优先、血量优先），调节机器人的不同参数（弹速，射频，热量上限等），并采用了简单的枪口热量限制。
6. 通过遥控器和键鼠控制

## 缺陷与目标
1. 没有超级电容，飞坡困难
2. 几乎不具备自瞄能力
3. 非常容易卡弹
4. 不具备单发能力，不具备打符能力（单发程序在24赛季已完成）
5. 陀螺仪yaw会漂
5. 图传链路
## 接线图

####  底盘C板

- CAN1 :      板间通讯
- CAN2：    Yaw轴电机，底盘3508电机
- USART6:  接受裁判系统数据
####  云台C板

- CAN1 :      板间通讯
- CAN2：    Pitch轴电机，摩擦轮3508，弹仓盖2006，拨盘电机2006
- USART1:  与MiniPC通讯

![五号步兵接线图](img\五号步兵接线图.png)

## 控制代码

### 1、底盘C板

功能实现主要在以下四个文件夹中
1.__/AlgorithmLayer__    一些控制函数
2.__/DriverLayer__           硬件自定义配置
3.__/PotocaLayer__          通信函数相关
4.__/RTOS Task__              各种任务

#### /AlgorithmLayer

- CRC.c     CRC校验
- pid.c       PID计算

#### /DriverLayer

- drv_can.c          自定义 CAN通信相关（用于控制电机，上下板间CAN通信）
- drv_usart.c        自定义USART相关（遥控器数据接收，裁判系统数据接收）

#### /PotocaLayer

- rc_potocal.c       遥控器解算函数
- judge.c               读取裁判系统数据

#### /RTOS Task

- Chassis_task.c  底盘控制任务（基本运动、功率限制、斜坡启动）
- Yaw_task.c        云台YAW轴控制
- UI_task.c           UI绘制
- INS_task.c         陀螺仪解算
- super_cap.c     超级电容

#### stm32f4xx_it.c

注意：将中断调用(除can)写进了stm32f4xx_it.c函数

### 2、云台C板

功能实现主要在以下三个文件夹中

1.__/DriverLayer__           硬件自定义配置
2.__/PotocaLayer__          通信函数相关
3.__/RTOS Task__              各种任务

#### /DriverLayer

- Can_user.c          自定义 CAN通信相关（用于控制电机，上下C板间CAN通信）

#### /PtocaLayer

- remote_control.c        遥控器数据接收函数
- MX_FREERTOS_init.c  自定义的freertos初始化函数
- struct_typedef.h          数据结构体定义

#### /RTOS Task

- Friction_task.c    发射控制任务（连续发射，弹仓盖开关，热量限制，弹速设置等）
- Pitch_task.c         云台Pitch轴控制
- INS_task.c            陀螺仪解算

#### stm32f4xx_it.c 

注意：将中断调用(除can)写进了stm32f4xx_it.c函数。

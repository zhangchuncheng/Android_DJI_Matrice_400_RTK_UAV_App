# 大疆MSDK上云API对接项目说明文档

## 1. 项目概述

本项目基于大疆MSDK V5官方Demo，实现了通过遥控器接入M400无人机和第三方机场，使现有Web平台无需改动代码即可通过上云API指令控制无人机和第三方机场。

### 核心功能
- MQTT消息订阅与解析，将Web平台下发的上云API指令转换为MSDK控制指令
- 无人机执行状态包装成上云API事件通过MQTT上报
- 支持第三方机场的MQTT消息控制
- 三个功能页面：状态信息页面、调试控制页面、配置页面

## 2. 项目结构

```
android-sdk-v5-sample/
├── src/main/java/dji/sampleV5/aircraft/
│   ├── mqtt/                    # MQTT相关模块
│   │   ├── CloudApiService.kt   # 上云API服务（核心）
│   │   ├── MqttManager.kt       # MQTT连接管理
│   │   ├── MqttTopicConstants.kt # MQTT主题常量
│   │   └── MqttMessageHandler.kt # MQTT消息处理器接口
│   ├── config/                  # 配置管理
│   │   ├── ConfigManager.kt     # 配置管理器
│   │   └── Config.kt            # 配置数据类
│   ├── pages/                   # 页面模块
│   │   ├── StatusFragment.kt    # 状态信息页面
│   │   ├── DebugFragment.kt     # 调试控制页面
│   │   └── ConfigFragment.kt    # 配置页面
│   ├── models/                  # 数据模型
│   │   └── BasicAircraftControlVM.kt # 无人机控制ViewModel
│   └── utils/                   # 工具类
├── src/main/res/layout/         # 布局文件
│   ├── frag_status_page.xml     # 状态页面布局
│   ├── frag_debug_page.xml      # 调试页面布局
│   └── frag_config_page.xml     # 配置页面布局
└── src/main/res/values/         # 资源文件
```

## 3. 核心文件说明

### 3.1 CloudApiService.kt

**位置**: [CloudApiService.kt](file:///E:/project/Mobile-SDK-Android-V5/SampleCode-V5/android-sdk-v5-sample/src/main/java/dji/sampleV5/aircraft/mqtt/CloudApiService.kt)

**功能**: 上云API服务核心类，负责处理MQTT消息与MSDK指令之间的转换

**主要方法**:

| 方法名 | 功能说明 |
|--------|----------|
| `connectMqtt()` | 连接MQTT服务器 |
| `disconnectMqtt()` | 断开MQTT连接 |
| `sendDeviceOnline()` | 发送设备上线消息 |
| `sendDeviceOffline()` | 发送设备离线消息 |
| `sendTopologyUpdate()` | 发送拓扑更新消息 |
| `sendEvent()` | 发送事件消息 |
| `sendServiceReply()` | 发送服务响应消息 |
| `handleServiceRequest()` | 处理服务请求消息 |
| `subscribeDeviceState()` | 订阅设备状态 |

**支持的上云API方法**:

| 方法名 | 功能说明 |
|--------|----------|
| `flight_authority_grab` | 抢夺飞行控制权 |
| `payload_authority_grab` | 抢夺负载控制权 |
| `drc_mode_enter` | 进入DRC模式 |
| `drc_mode_exit` | 退出DRC模式 |
| `takeoff_to_point` | 一键起飞到目标点 |
| `fly_to_point` | 飞向目标点 |
| `goto_point` | 飞向指定点 |
| `land` | 自动降落 |
| `return_home` | 返航 |
| `start_live_stream` | 开始直播 |
| `stop_live_stream` | 停止直播 |
| `gimbal_move` | 云台转动 |
| `gimbal_reset` | 云台复位 |
| `take_photo` | 拍照 |
| `start_video` | 开始录像 |
| `stop_video` | 停止录像 |
| `upload_wayline` | 航线上传 |
| `start_wayline` | 启动航线任务 |
| `pause_wayline` | 暂停航线任务 |
| `resume_wayline` | 恢复航线任务 |
| `stop_wayline` | 停止航线任务 |
| `spotlight_mode` | 探照灯模式 |
| `start_interest_point` | 兴趣点环绕 |
| `check_upgrade` | 检查升级 |
| `start_upgrade` | 开始升级 |
| `list_media` | 获取媒体列表 |
| `download_media` | 下载媒体文件 |
| `megaphone_play` | 喊话器播放 |
| `megaphone_stop` | 停止喊话 |
| `megaphone_set_volume` | 设置喊话器音量 |
| `set_camera_mode` | 设置相机模式 |
| `set_gimbal_mode` | 设置云台模式 |

### 3.2 MqttManager.kt

**位置**: [MqttManager.kt](file:///E:/project/Mobile-SDK-Android-V5/SampleCode-V5/android-sdk-v5-sample/src/main/java/dji/sampleV5/aircraft/mqtt/MqttManager.kt)

**功能**: MQTT连接管理，负责连接、断开、发布、订阅等操作

**主要方法**:

| 方法名 | 功能说明 |
|--------|----------|
| `connect()` | 连接MQTT服务器 |
| `disconnect()` | 断开MQTT连接 |
| `publish()` | 发布MQTT消息 |
| `subscribeProductTopics()` | 订阅设备主题 |
| `subscribeThirdPartyAirportTopics()` | 订阅第三方机场主题 |
| `setMqttMessageListener()` | 设置消息监听器 |

### 3.3 ConfigManager.kt

**位置**: [ConfigManager.kt](file:///E:/project/Mobile-SDK-Android-V5/SampleCode-V5/android-sdk-v5-sample/src/main/java/dji/sampleV5/aircraft/config/ConfigManager.kt)

**功能**: 配置管理，负责配置的读写和持久化

**配置项**:

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `mqttHost` | MQTT服务器地址 | 192.168.2.110 |
| `mqttPort` | MQTT端口 | 1883 |
| `mqttUsername` | MQTT用户名 | nichst |
| `mqttPassword` | MQTT密码 | Nichst.123 |
| `mqttClientId` | MQTT客户端ID | mqtt-client-{random} |
| `productSn` | 产品序列号 | - |
| `aircraftModel` | 无人机型号 | - |
| `thirdPartyAirportEnabled` | 是否启用第三方机场 | false |
| `thirdPartyAirportTopic` | 第三方机场MQTT主题 | - |

## 4. 页面说明

### 4.1 状态信息页面

**位置**: [StatusFragment.kt](file:///E:/project/Mobile-SDK-Android-V5/SampleCode-V5/android-sdk-v5-sample/src/main/java/dji/sampleV5/aircraft/pages/StatusFragment.kt)

**功能**: 显示无人机连接状态和基本信息

**显示内容**:
- 连接状态（对频、已连接、未连接）
- 设备型号和产品序列号
- 飞行模式
- 电池电量
- 当前位置（经纬度）
- 高度、速度、航向
- 飞行时间

### 4.2 调试控制页面

**位置**: [DebugFragment.kt](file:///E:/project/Mobile-SDK-Android-V5/SampleCode-V5/android-sdk-v5-sample/src/main/java/dji/sampleV5/aircraft/pages/DebugFragment.kt)

**功能**: 提供调试控制按钮和日志显示

**控制按钮**:
- 飞行控制：起飞、降落、返航
- 视频直播：开始直播、停止直播
- MQTT控制：连接MQTT、断开MQTT、发送上线、更新拓扑
- 控制权管理：抢夺控制权、释放控制权、抢夺负载控制、释放负载控制

**日志显示**:
- 操作日志：显示操作记录
- MQTT报文：显示收到和发送的MQTT消息（彩色区分）

### 4.3 配置页面

**位置**: [ConfigFragment.kt](file:///E:/project/Mobile-SDK-Android-V5/SampleCode-V5/android-sdk-v5-sample/src/main/java/dji/sampleV5/aircraft/pages/ConfigFragment.kt)

**功能**: 配置MQTT和设备信息

**配置项**:
- MQTT服务器地址和端口
- MQTT用户名和密码
- MQTT客户端ID
- 产品序列号
- 无人机型号
- 第三方机场配置

## 5. MQTT主题说明

### 5.1 订阅主题

| 主题模式 | 说明 |
|----------|------|
| `thing/product/{productSn}/services` | 服务请求 |
| `thing/product/{productSn}/property/set` | 属性设置 |
| `thing/product/{productSn}/drc/down` | DRC下行消息 |

### 5.2 发布主题

| 主题模式 | 说明 |
|----------|------|
| `sys/product/{productSn}/status` | 设备状态/拓扑更新 |
| `thing/product/{productSn}/osd` | OSD状态上报 |
| `thing/product/{productSn}/events` | 事件上报 |
| `thing/product/{productSn}/services_reply` | 服务响应 |
| `thing/product/{productSn}/property/set_reply` | 属性设置响应 |

### 5.3 消息格式

**服务请求**:
```json
{
  "tid": "事务ID",
  "bid": "业务ID",
  "method": "方法名",
  "params": {
    "参数1": "值1",
    "参数2": "值2"
  }
}
```

**服务响应**:
```json
{
  "tid": "事务ID",
  "bid": "业务ID",
  "method": "方法名",
  "data": {
    "result": 0
  },
  "timestamp": 1620000000000
}
```

**事件上报**:
```json
{
  "timestamp": 1620000000000,
  "event_type": "事件类型",
  "data": {
    "属性1": "值1"
  }
}
```

**OSD上报**:
```json
{
  "timestamp": 1620000000000,
  "product_sn": "产品序列号",
  "device_type": "aircraft",
  "connection_status": "connected",
  "flight_mode": "GPS_ATTI",
  "battery_level": 85,
  "altitude": 10.5,
  "latitude": 30.0,
  "longitude": 120.0,
  "speed": 5.0,
  "heading": 90.0,
  "flight_time": 300
}
```

## 6. 第三方机场集成

### 6.1 配置

在配置页面启用第三方机场，并设置第三方机场的MQTT主题。

### 6.2 消息流程

1. Web平台下发指令到App的MQTT服务主题
2. App解析消息，识别为第三方机场指令
3. App将指令转发到第三方机场的MQTT主题
4. 第三方机场响应消息后，App接收并按照上云API格式返回给Web平台

### 6.3 支持的第三方机场指令

所有服务请求方法都支持转发到第三方机场。

## 7. 模拟器功能

项目已集成MSDK模拟器功能，可通过SimulatorFragment启用：

**功能**:
- 启用/禁用模拟器
- 设置模拟坐标
- 设置GPS信号数量

**使用步骤**:
1. 打开模拟器页面
2. 设置模拟经纬度
3. 设置GPS信号数量（建议10-12）
4. 点击启用模拟器
5. 设备会模拟连接到无人机

## 8. 使用步骤

### 8.1 首次配置

1. 打开App，进入配置页面
2. 填写MQTT服务器配置（地址、端口、用户名、密码）
3. 填写产品序列号和无人机型号
4. 如需接入第三方机场，启用并设置主题
5. 点击保存配置

### 8.2 连接MQTT

1. 进入调试页面
2. 点击"连接MQTT"按钮
3. 查看MQTT日志确认连接成功

### 8.3 设备上线

1. 在调试页面点击"发送上线"
2. Web平台会收到设备上线消息
3. 设备会自动上报拓扑信息

### 8.4 接收指令

设备连接成功后，Web平台下发的上云API指令会自动解析并执行：

**示例指令**:
```json
{
  "tid": "123456",
  "bid": "abcdef",
  "method": "takeoff_to_point",
  "params": {
    "target_latitude": 30.0,
    "target_longitude": 120.0,
    "target_height": 50,
    "flight_id": "flight-001"
  }
}
```

### 8.5 状态上报

设备会自动定时上报OSD状态（每3秒一次），包含：
- 位置信息（经纬度、高度）
- 飞行状态（飞行模式、速度、航向）
- 电池状态（电量）
- 设备信息（产品类型、连接状态）

## 9. 开发注意事项

### 9.1 MQTT消息格式

所有消息必须包含 `tid` 和 `bid` 字段，参数统一放在 `params` 字段中。

### 9.2 控制权管理

执行飞行控制指令前，必须先抢夺飞行控制权：
1. 发送 `flight_authority_grab` 指令
2. 成功后才能执行起飞、降落等操作
3. 完成后发送 `drc_mode_exit` 释放控制权

### 9.3 日志调试

调试页面提供了完整的日志功能：
- 操作日志：显示按钮点击和操作结果
- MQTT日志：显示所有收发的MQTT报文

### 9.4 错误处理

所有MSDK调用都有完整的错误处理：
- 成功时发送服务响应和事件
- 失败时发送错误信息和事件
- 错误信息包含具体的错误描述

## 10. 编译与运行

### 10.1 编译环境

- Android Studio Arctic Fox 或更高版本
- Gradle 7.0 或更高版本
- Java 11

### 10.2 编译命令

```bash
./gradlew compileDebugKotlin
./gradlew assembleDebug
```

### 10.3 安装到遥控器

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## 11. 常见问题

### Q1: 日志不显示

**解决方案**:
- 确保布局文件中 `tv_log` 和 `tv_mqtt_log` 已正确定义
- 确保 `addLog()` 和 `addMqttLog()` 方法在主线程中执行
- 检查MQTT连接状态，只有连接成功后才会接收消息

### Q2: 指令不执行

**解决方案**:
- 检查MQTT连接状态
- 检查消息格式是否正确（必须包含tid、bid、method字段）
- 检查设备是否已连接（可在状态页面查看）
- 检查是否已抢夺控制权

### Q3: 编译错误

**解决方案**:
- 检查MSDK版本是否正确
- 检查依赖是否完整
- 检查KeyTools导入路径是否正确

### Q4: 第三方机场无响应

**解决方案**:
- 检查第三方机场主题配置是否正确
- 检查第三方机场是否已连接到MQTT服务器
- 检查消息格式是否符合第三方机场协议

## 12. 更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2026-07 | v1.0 | 初始版本，实现核心功能 |
| 2026-07 | v1.1 | 添加模拟器功能、控制权管理 |
| 2026-07 | v1.2 | 修复日志显示、完善第三方机场集成 |

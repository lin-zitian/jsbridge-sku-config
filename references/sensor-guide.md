# 传感器(Sensor)配置指南

## 设备类型
- **类型标识**: `devices.types.sensor`
- **适用设备**: 人体感应传感器、空气质量传感器、CO2传感器

## 参考 SKU
- **H5127**: 人体感应传感器

## 常用 Instance 能力

### 1. 传感器数据(只读)
- `sensorTemperature`: 传感器温度
- `sensorHumidity`: 传感器湿度
- `carbonDioxideConcentration`: 二氧化碳浓度
- `airQuality`: 空气质量

### 2. 事件告警
- `bodyAppearedEvent`: 人体出现事件

### 3. 系统功能
- `heartbeat`: 心跳包
- `bindDevice`: 设备绑定
- `sensorQuery`: 传感器数据查询

## BLE 指令配置

### 一致性指令(所有传感器通用)

#### bodyAppearedEvent - 人体出现事件
```javascript
{
  instance: InstanceEnum.bodyAppearedEvent.value,
  type: 'devices.capabilities.event',
  ctrlPlatformSupported: ['openApi'],
  alarmType: 56,
  eventState: {
    options: [
      { name: 'bodyAppeared', value: 1, message: 'Body appeared' }
    ]
  }
}
```

#### sensorTemperature - 传感器温度
```javascript
{
  instance: InstanceEnum.sensorTemperature.value,
  type: 'devices.capabilities.property',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [],
    read: ['aa', '10']
  }
}
```

#### sensorHumidity - 传感器湿度
```javascript
{
  instance: InstanceEnum.sensorHumidity.value,
  type: 'devices.capabilities.property',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [],
    read: ['aa', '10']
  }
}
```

#### carbonDioxideConcentration - CO2浓度
```javascript
{
  instance: InstanceEnum.carbonDioxideConcentration.value,
  type: 'devices.capabilities.property',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [],
    read: ['aa', '10']
  }
}
```

#### airQuality - 空气质量
```javascript
{
  instance: InstanceEnum.airQuality.value,
  type: 'devices.capabilities.property',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [],
    read: ['aa', '10']
  }
}
```

#### heartbeat - 心跳包
```javascript
{
  instance: 'heartbeat',
  type: 'devices.capabilities.sys_read',
  ctrlPlatformSupported: ['pad'],
  action: {
    write: [],
    read: ['aa', '01']
  }
}
```

#### bindDevice - 设备绑定
```javascript
{
  instance: 'bindDevice',
  type: 'devices.capabilities.bind_device',
  ctrlPlatformSupported: ['pad'],
  action: {
    write: [],
    read: ['aa', '08']
  }
}
```

#### sensorQuery - 传感器查询
```javascript
{
  instance: 'sensorQuery',
  type: 'devices.capabilities.sensor_query',
  ctrlPlatformSupported: ['pad'],
  action: {
    write: [],
    read: ['aa', '10']
  }
}
```

## 配置流程

### Step 1: 确认基础信息
询问用户:
1. SKU 编号
2. goodsType(产品类型)
3. 传感器类型(人体感应/空气质量/CO2等)

### Step 2: 确认传感器类型
询问用户:
1. 是否支持人体感应? (bodyAppearedEvent)
2. 是否支持温度传感器? (sensorTemperature)
3. 是否支持湿度传感器? (sensorHumidity)
4. 是否支持CO2浓度? (carbonDioxideConcentration)
5. 是否支持空气质量? (airQuality)

### Step 3: 配置能力
传感器配置非常简单:
- 根据传感器类型添加相应的 instance
- 添加系统功能(heartbeat/bindDevice/sensorQuery)

## 常见配置模式

### 模式 1: 人体感应传感器
- bodyAppearedEvent
- heartbeat
- bindDevice
- sensorQuery

### 模式 2: 环境传感器
- sensorTemperature
- sensorHumidity
- heartbeat
- bindDevice
- sensorQuery

### 模式 3: 空气质量传感器
- carbonDioxideConcentration
- airQuality
- sensorTemperature
- sensorHumidity
- heartbeat
- bindDevice
- sensorQuery

## 注意事项

1. **只读设备**: 传感器通常只有读取功能,没有控制功能
2. **事件告警**: 人体感应等事件需要配置 alarmType 和 eventState
3. **无 payloadDefine**: 传感器能力不需要 payloadDefine
4. **平台支持**: 
   - 传感器数据和事件支持 openApi
   - 系统功能只支持 pad
5. **指令地址**: 
   - 传感器查询: 0x10
   - 心跳: 0x01
   - 绑定: 0x08
6. **无 BLE 配置需求**: 所有指令都是标准的,不需要用户提供 BLE 配置
7. **alarmType**: 不同事件有不同的 alarmType,需要确认

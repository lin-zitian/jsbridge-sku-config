# 温湿度计(Thermometer)配置指南

## 设备类型
- **类型标识**: `devices.types.thermometer`
- **适用设备**: 温湿度计、温度传感器

## 参考 SKU
- **H5110**: 标准温湿度计
- **H5111**: 温湿度计变体

## 常用 Instance 能力

### 1. 传感器数据(只读)
- `sensorTemperature`: 传感器温度
- `sensorHumidity`: 传感器湿度

### 2. 校准功能
- `temCalibration`: 温度校准
- `humCalibration`: 湿度校准

### 3. 系统功能
- `heartbeat`: 心跳包
- `bindDevice`: 设备绑定
- `sensorQuery`: 传感器数据查询

## BLE 指令配置

### 一致性指令(所有温湿度计通用)

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

**注意**: 温湿度计通常只有读取功能,没有写入功能

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

#### temCalibration - 温度校准
```javascript
{
  instance: 'temCalibration',
  type: 'devices.capabilities.tem_calibration',
  ctrlPlatformSupported: ['pad'],
  action: {
    write: [],
    read: ['aa', '1e']
  }
}
```

**注意**: 校准功能通常只在 pad 平台使用

#### humCalibration - 湿度校准
```javascript
{
  instance: 'humCalibration',
  type: 'devices.capabilities.hum_calibration',
  ctrlPlatformSupported: ['pad'],
  action: {
    write: [],
    read: ['aa', '1e']
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
3. 是否只测温度,还是温湿度都测?

### Step 2: 确认传感器类型
询问用户:
1. 是否支持温度传感器? (sensorTemperature)
2. 是否支持湿度传感器? (sensorHumidity)

### Step 3: 确认校准功能
询问用户:
1. 是否支持温度校准? (temCalibration)
2. 是否支持湿度校准? (humCalibration)

### Step 4: 配置能力
温湿度计配置非常简单:
- 添加 sensorTemperature (如果支持温度)
- 添加 sensorHumidity (如果支持湿度)
- 添加校准功能(如果需要)
- 添加系统功能(heartbeat/bindDevice/sensorQuery)

## 常见配置模式

### 模式 1: 纯温度计
- sensorTemperature
- temCalibration
- heartbeat
- bindDevice
- sensorQuery

### 模式 2: 温湿度计
- sensorTemperature
- sensorHumidity
- temCalibration
- humCalibration
- heartbeat
- bindDevice
- sensorQuery

## 注意事项

1. **只读设备**: 温湿度计通常只有读取功能,没有控制功能
2. **无 payloadDefine**: 传感器能力不需要 payloadDefine
3. **平台支持**: 
   - 传感器数据(sensorTemperature/sensorHumidity)支持 openApi
   - 校准和系统功能只支持 pad
4. **指令地址**: 
   - 传感器查询: 0x10
   - 校准: 0x1e
   - 心跳: 0x01
   - 绑定: 0x08
5. **无 BLE 配置需求**: 所有指令都是标准的,不需要用户提供 BLE 配置

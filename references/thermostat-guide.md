# 恒温器(Thermostat)配置指南

## 设备类型
- **类型标识**: `devices.types.thermostat`
- **适用设备**: 温控器、恒温器、智能温控面板

## 参考 SKU
暂无参考 SKU,需要根据实际项目补充

## 常用 Instance 能力

### 1. 基础控制
- `powerSwitch`: 电源开关
- `targetTemperature`: 目标温度
- `rangeTemperature`: 温度范围控制

### 2. 传感器数据
- `sensorTemperature`: 当前温度(只读)

### 3. 工作模式
- `workMode`: 工作模式(加热/制冷/自动/关闭)

### 4. 开关类
- `thermostatToggle`: 恒温开关

## BLE 指令配置

### 预期指令结构

#### powerSwitch - 电源开关
```javascript
action: {
  write: [{ bleDefine: ['33', '01'], valIndex: [2] }],
  read: ['aa', '01']
},
payloadDefine: { cmdType: 'multiSync' }
```

#### targetTemperature - 目标温度
```javascript
action: {
  write: [{ bleDefine: ['3a', '05', '03', '00', '32', '00'], valIndex: [3, 4, 5] }],
  read: ['aa', '05']
},
parameters: {
  dataType: 'STRUCT',
  fields: [
    {
      fieldName: 'temperature',
      dataType: 'INTEGER',
      range: { min: 10, max: 35, precision: 0.5 }
    },
    {
      fieldName: 'unit',
      dataType: 'ENUM',
      defaultValue: 'Celsius',
      options: [
        { name: 'Celsius', value: 'Celsius' },
        { name: 'Fahrenheit', value: 'Fahrenheit' }
      ]
    }
  ]
}
```

**注意**: 
- 温度范围通常为10-35°C
- 精度可能为0.5°C或1°C
- 支持华氏度/摄氏度切换

#### rangeTemperature - 温度范围控制
```javascript
action: {
  write: [{ bleDefine: ['3a', '05', '03', 'FF', '32', '00', '00', '00', '00'], valIndex: [3, 4, 5, 6, 7, 8] }],
  read: ['aa', '05']
},
parameters: {
  dataType: 'STRUCT',
  fields: [
    {
      fieldName: 'lowerSetpoint',
      dataType: 'INTEGER',
      range: { min: 10, max: 35, precision: 0.5 }
    },
    {
      fieldName: 'upperSetpoint',
      dataType: 'INTEGER',
      range: { min: 10, max: 35, precision: 0.5 }
    },
    {
      fieldName: 'unit',
      dataType: 'ENUM',
      defaultValue: 'Celsius'
    }
  ]
}
```

#### workMode - 工作模式
```javascript
action: {
  write: [{ bleDefine: ['3a', '05'], valIndex: [2, 3] }],
  read: ['aa', '05']
},
parameters: {
  dataType: 'STRUCT',
  fields: [
    {
      fieldName: 'workMode',
      options: [
        { name: 'Heat', value: 1 },
        { name: 'Cool', value: 2 },
        { name: 'Auto', value: 3 },
        { name: 'Off', value: 0 }
      ]
    },
    {
      fieldName: 'modeValue',
      options: [
        { name: 'Heat', defaultValue: 0 },
        { name: 'Cool', defaultValue: 0 },
        { name: 'Auto', defaultValue: 0 },
        { name: 'Off', defaultValue: 0 }
      ]
    }
  ]
}
```

#### sensorTemperature - 当前温度
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

## 配置流程

### Step 1: 确认基础信息
询问用户:
1. SKU 编号
2. goodsType(产品类型)
3. 恒温器类型(加热/制冷/双向)

### Step 2: 确认温度控制方式
询问用户:
1. 温度控制方式? (单点/范围)
2. 温度范围? (默认10-35°C)
3. 温度精度? (0.5°C/1°C)
4. 是否支持华氏度?

### Step 3: 确认工作模式
询问用户:
1. 支持哪些工作模式? (Heat/Cool/Auto/Off)
2. 是否只支持加热?
3. 是否支持制冷?
4. 是否支持自动模式?

### Step 4: 确认传感器
询问用户:
1. 是否有温度传感器? (sensorTemperature)
2. 传感器精度?

### Step 5: BLE 指令配置
需要用户提供具体的 BLE 指令配置

## 常见配置模式

### 模式 1: 基础恒温器
- powerSwitch
- targetTemperature
- sensorTemperature
- workMode (Heat/Off)

### 模式 2: 双向恒温器
- 基础恒温器 +
- workMode (Heat/Cool/Auto/Off)

### 模式 3: 高级恒温器
- 双向恒温器 +
- rangeTemperature (范围控制)
- thermostatToggle

## 注意事项

1. **cmdType**: 恒温器通常使用 `multiSync`
2. **温度范围**: 通常为10-35°C,需要确认
3. **温度精度**: 可能为0.5°C或1°C,需要确认
4. **温度单位**: 通常支持华氏度/摄氏度切换
5. **工作模式**: 确认是单向(只加热)还是双向(加热+制冷)
6. **传感器**: 恒温器通常有温度传感器用于反馈
7. **指令地址**: 温度相关指令通常使用 0x05
8. **参考实现**: 可以参考取暖器(H7138)的温度控制实现

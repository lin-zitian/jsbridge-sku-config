# 水壶(Kettle)配置指南

## 设备类型
- **类型标识**: `devices.types.kettle`
- **适用设备**: 电热水壶、保温壶、智能水壶

## 参考 SKU
暂无参考 SKU,需要根据实际项目补充

## 常用 Instance 能力

### 1. 基础控制
- `powerSwitch`: 电源开关
- `temperature`: 温度调节
- `targetTemperature`: 目标温度

### 2. 开关类
- `thermostatToggle`: 恒温开关

### 3. 工作模式
- `workMode`: 工作模式(加热/保温/快速加热等)

### 4. 事件告警
- `waterFullEvent`: 水满事件
- `lackWaterEvent`: 缺水事件

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

#### temperature - 温度调节
```javascript
action: {
  write: [{ bleDefine: ['3a', '05', '03', '00', '00'], valIndex: [3, 4] }],
  read: ['aa', '05']
},
parameters: {
  dataType: 'STRUCT',
  fields: [
    {
      fieldName: 'temperature',
      dataType: 'INTEGER',
      range: { min: 40, max: 100, precision: 1 }
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
- 温度范围通常为40-100°C
- 可能支持华氏度/摄氏度切换
- 有些型号可能不支持 autoStop 字段

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
        { name: 'Keep Warm', value: 2 },
        { name: 'Quick Heat', value: 3 }
      ]
    },
    {
      fieldName: 'modeValue',
      options: [
        { name: 'Heat', defaultValue: 0 },
        { name: 'Keep Warm', defaultValue: 0 },
        { name: 'Quick Heat', defaultValue: 0 }
      ]
    }
  ]
}
```

#### lackWaterEvent - 缺水事件
```javascript
{
  instance: InstanceEnum.lackWaterEvent.value,
  type: 'devices.capabilities.event',
  ctrlPlatformSupported: ['openApi'],
  alarmType: 51,
  eventState: {
    options: [
      { name: 'lack', value: 1, message: 'Lack of Water' }
    ]
  }
}
```

#### waterFullEvent - 水满事件
```javascript
{
  instance: InstanceEnum.waterFullEvent.value,
  type: 'devices.capabilities.event',
  ctrlPlatformSupported: ['openApi'],
  alarmType: 52,
  eventState: {
    options: [
      { name: 'waterFull', value: 1, message: 'Water Full' }
    ]
  }
}
```

## 配置流程

### Step 1: 确认基础信息
询问用户:
1. SKU 编号
2. goodsType(产品类型)
3. 水壶类型(电热水壶/保温壶等)

### Step 2: 确认温度控制
询问用户:
1. 温度调节范围? (默认40-100°C)
2. 是否支持华氏度?
3. 是否支持目标温度设置?

### Step 3: 确认工作模式
询问用户:
1. 支持哪些工作模式? (加热/保温/快速加热等)
2. 是否支持恒温功能?

### Step 4: 确认事件告警
询问用户:
1. 是否支持缺水检测?
2. 是否支持水满检测?
3. 是否有其他告警事件?

### Step 5: BLE 指令配置
需要用户提供具体的 BLE 指令配置

## 常见配置模式

### 模式 1: 基础水壶
- powerSwitch
- temperature
- lackWaterEvent

### 模式 2: 智能水壶
- 基础水壶 +
- workMode (Heat/Keep Warm)
- thermostatToggle
- waterFullEvent

### 模式 3: 高级水壶
- 智能水壶 +
- targetTemperature
- 多种工作模式

## 注意事项

1. **cmdType**: 水壶通常使用 `multiSync`
2. **温度范围**: 通常为40-100°C,需要确认
3. **温度单位**: 确认是否支持华氏度/摄氏度切换
4. **autoStop**: 有些型号可能不支持 autoStop 字段
5. **事件告警**: 缺水/水满事件的 alarmType 需要确认
6. **指令地址**: 温度相关指令通常使用 0x05
7. **参考实现**: 由于暂无参考 SKU,需要参考取暖器(H7138)的温度控制实现

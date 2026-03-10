# 取暖器(Heater)配置指南

## 设备类型
- **类型标识**: `devices.types.heater`
- **适用设备**: 电暖器、暖风机、油汀

## 参考 SKU
- **H7138**: 标准取暖器，支持温度调节、工作模式、范围控制

## 常用 Instance 能力

### 1. 基础控制
- `powerSwitch`: 电源开关
- `workMode`: 工作模式(档位/风扇/自动/睡眠)
- `gearMode`: 档位调节(Low/Medium/High)

### 2. 温度控制
- `targetTemperature`: 目标温度(单点控制)
- `rangeTemperature`: 温度范围控制(上下限)
- `sliderTemperature`: 滑动温度调节
- `sensorTemperature`: 传感器温度(只读)

### 3. 开关类
- `thermostatToggle`: 恒温开关
- `oscillationToggle`: 摇头开关

### 4. 延时功能
- `powerOffDuration`: 电源延时关

## BLE 指令配置

### 一致性指令(所有取暖器通用)

#### powerSwitch - 电源开关
```javascript
action: {
  write: [{ bleDefine: ['33', '01'], valIndex: [2] }],
  read: ['aa', '01']
},
payloadDefine: { cmdType: 'multiSync' }
```

### 可能变化的指令

#### targetTemperature - 目标温度
**H7138 结构**:
```javascript
action: {
  write: [{ bleDefine: ['3a', '05', '03', '00', '32', '00'], valIndex: [3, 4, 5] }],
  read: ['aa', '05']
},
parameters: {
  dataType: 'STRUCT',
  fields: [
    {
      fieldName: 'autoStop',
      dataType: 'ENUM',
      defaultValue: 0,
      options: [
        { name: 'Auto Stop', value: 1 },
        { name: 'Maintain', value: 0 }
      ]
    },
    {
      fieldName: 'temperature',
      dataType: 'INTEGER',
      range: { min: 5, max: 30, precision: 1 }
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
- 温度范围因型号而异(常见: 5-30°C, 10-35°C)
- 有些型号支持华氏度/摄氏度切换
- autoStop 字段表示达到目标温度后是否自动停止

#### rangeTemperature - 温度范围控制
**H7138 结构**:
```javascript
action: {
  write: [{ bleDefine: ['3a', '05', '03', 'FF', '32', '00', '00', '00', '00'], valIndex: [3, 4, 5, 6, 7, 8] }],
  read: ['aa', '05']
},
parameters: {
  dataType: 'STRUCT',
  fields: [
    {
      fieldName: 'autoStop',
      dataType: 'ENUM',
      defaultValue: 0
    },
    {
      fieldName: 'gearMode',
      dataType: 'ENUM',
      defaultValue: 1,
      options: [
        { name: 'Low', value: 1 },
        { name: 'Medium', value: 2 },
        { name: 'High', value: 3 }
      ]
    },
    {
      fieldName: 'lowerSetpoint',
      dataType: 'INTEGER',
      range: { min: 5, max: 30, precision: 1 }
    },
    {
      fieldName: 'upperSetpoint',
      dataType: 'INTEGER',
      range: { min: 5, max: 30, precision: 1 }
    },
    {
      fieldName: 'unit',
      dataType: 'ENUM',
      defaultValue: 'Celsius'
    }
  ]
}
```

**注意**: 范围控制允许设置温度上下限,配合档位使用

#### workMode - 工作模式
**H7138 结构**:
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
        { name: 'gearMode', value: 1 },
        { name: 'Fan', value: 9 },
        { name: 'Auto', value: 3 },
        { name: 'Sleep', value: 5 }
      ]
    },
    {
      fieldName: 'modeValue',
      options: [
        { name: 'gearMode', options: [
          { name: 'Low', value: 1 },
          { name: 'Medium', value: 2 },
          { name: 'High', value: 3 }
        ]},
        { name: 'Fan', defaultValue: 0 },
        { name: 'Auto', defaultValue: 0 },
        { name: 'Sleep', defaultValue: 0 }
      ]
    }
  ]
}
```

**注意**: 
- gearMode 模式需要指定档位(Low/Medium/High)
- Fan 模式表示只送风不加热
- Auto/Sleep 模式通常不需要 modeValue

## 配置流程

### Step 1: 确认基础信息
询问用户:
1. SKU 编号
2. goodsType(产品类型)
3. 取暖器类型(暖风机/油汀/电暖器等)

### Step 2: 确认温度控制方式
询问用户:
1. 温度控制方式? (单点/范围)
2. 温度范围? (默认5-30°C)
3. 是否支持华氏度?
4. 是否支持 autoStop(达到温度自动停止)?

### Step 3: 确认工作模式
询问用户:
1. 支持哪些工作模式? (gearMode/Fan/Auto/Sleep等)
2. 档位模式有几档? (Low/Medium/High 或更多)
3. 是否支持风扇模式(只送风)?
4. 是否支持自动模式?
5. 是否支持睡眠模式?

### Step 4: 确认其他功能
询问用户:
1. 是否支持恒温功能? (thermostatToggle)
2. 是否支持摇头功能? (oscillationToggle)
3. 是否支持延时关机?
4. 是否有温度传感器? (sensorTemperature)

### Step 5: BLE 指令配置
- powerSwitch 使用标准配置
- targetTemperature/rangeTemperature 需要根据温度范围配置
- workMode 需要根据支持的模式和档位配置

## 常见配置模式

### 模式 1: 基础取暖器
- powerSwitch
- gearMode (Low/Medium/High)
- powerOffDuration

### 模式 2: 智能取暖器
- 基础取暖器 +
- targetTemperature (单点温度控制)
- workMode (gearMode/Auto/Sleep)
- sensorTemperature

### 模式 3: 高级取暖器
- 智能取暖器 +
- rangeTemperature (范围控制)
- thermostatToggle
- oscillationToggle

## 注意事项

1. **cmdType**: 取暖器通常使用 `multiSync`
2. **温度单位**: 确认是否支持华氏度/摄氏度切换
3. **温度范围**: 不同型号范围不同,需要确认
4. **autoStop**: 确认是否支持达到温度自动停止
5. **工作模式**: gearMode 需要指定具体档位
6. **范围控制**: rangeTemperature 需要配合 gearMode 使用
7. **指令地址**: 温度相关指令通常使用 0x05

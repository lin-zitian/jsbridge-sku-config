# 风扇(Fan)配置指南

## 设备类型
- **类型标识**: `devices.types.fan`
- **适用设备**: 电风扇、循环扇、塔扇

## 参考 SKU
- **H7100**: 标准风扇,支持摇头、档位调节

## 常用 Instance 能力

### 1. 基础控制
- `powerSwitch`: 电源开关
- `fanSpeed`: 风速调节
- `fanSpeedMode`: 风速档位模式
- `gearMode`: 档位调节

### 2. 开关类
- `oscillationToggle`: 摇头开关
- `fanOscillateToggle`: 风扇摇头开关
- `swingLeafToggle`: 摆叶开关
- `airDeflectorToggle`: 导流板开关
- `reverseAirflowToggle`: 反向出风开关

### 3. 模式控制
- `workMode`: 工作模式(Normal/Natural/Sleep/Auto)
- `deflectorSpeedMode`: 导流板速度
- `deflectorRange`: 导板摆动范围

### 4. 延时功能
- `powerOffDuration`: 电源延时关

## BLE 指令配置

### 一致性指令(所有风扇通用)

#### powerSwitch - 电源开关
```javascript
action: {
  write: [{ bleDefine: ['33', '01'], valIndex: [2] }],
  read: ['aa', '01']
},
payloadDefine: { cmdType: 'multiSync' }
```

#### oscillationToggle - 摇头开关
```javascript
action: {
  write: [{ bleDefine: ['33', '1f', '01'], valIndex: [3] }],
  read: ['aa', '1f', '01']
},
payloadDefine: { cmdType: 'multiSync' }
```

### 可能变化的指令

#### workMode - 工作模式
**常见结构**:
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
        { name: 'Normal', value: 1 },
        { name: 'Natural', value: 2 },
        { name: 'Sleep', value: 3 },
        { name: 'Auto', value: 4 }
      ]
    },
    {
      fieldName: 'modeValue',
      options: [
        { name: 'Normal', options: [{ value: 1 }, { value: 2 }, ...] },
        { name: 'Natural', defaultValue: 0 },
        { name: 'Sleep', defaultValue: 0 },
        { name: 'Auto', defaultValue: 0 }
      ]
    }
  ]
}
```

**注意**: 不同型号支持的模式和档位数量可能不同

#### fanSpeed - 风速调节
```javascript
action: {
  write: [{ bleDefine: ['3a', '05', '01'], valIndex: [3] }]
},
parameters: {
  dataType: 'INTEGER',
  range: { min: 1, max: 12, precision: 1 }
}
```

**注意**: 风速范围因型号而异(常见: 1-8, 1-12, 1-16)

#### fanSpeedMode - 风速档位
```javascript
parameters: {
  dataType: 'ENUM',
  options: [
    { name: 'Low', value: 1 },
    { name: 'Medium', value: 2 },
    { name: 'High', value: 3 }
  ]
}
```

**注意**: 档位数量和名称可能不同

## 配置流程

### Step 1: 确认基础信息
询问用户:
1. SKU 编号
2. goodsType(产品类型)
3. 风扇类型(塔扇/循环扇/落地扇等)

### Step 2: 确认风速控制
询问用户:
1. 风速调节方式? (连续调节/档位调节)
2. 如果是连续调节,范围是多少? (1-8/1-12/1-16等)
3. 如果是档位调节,有几档? (Low/Medium/High 或更多)

### Step 3: 确认工作模式
询问用户:
1. 支持哪些工作模式? (Normal/Natural/Sleep/Auto等)
2. Normal 模式有几个档位?
3. 是否支持自然风模式?
4. 是否支持睡眠模式?

### Step 4: 确认摇头功能
询问用户:
1. 是否支持摇头? (oscillationToggle)
2. 是否支持摆叶? (swingLeafToggle)
3. 是否支持导流板? (airDeflectorToggle)
4. 是否支持反向出风? (reverseAirflowToggle)

### Step 5: 确认其他功能
询问用户:
1. 是否支持延时关机?
2. 是否有其他特殊功能?

### Step 6: BLE 指令配置
- 基础指令(powerSwitch/oscillationToggle)使用标准配置
- workMode/fanSpeed 需要根据用户提供的信息配置

## 常见配置模式

### 模式 1: 基础风扇
- powerSwitch
- fanSpeed 或 fanSpeedMode
- oscillationToggle

### 模式 2: 智能风扇
- 基础风扇 +
- workMode (Normal/Natural/Sleep)
- powerOffDuration

### 模式 3: 高级风扇
- 智能风扇 +
- swingLeafToggle
- airDeflectorToggle
- deflectorSpeedMode
- deflectorRange

## 注意事项

1. **cmdType**: 风扇通常使用 `multiSync`
2. **风速控制**: 确认是连续调节还是档位调节
3. **摇头功能**: 不同型号可能使用不同的 instance
4. **工作模式**: Normal 模式的档位数量需要确认
5. **指令地址**: 风速相关指令通常使用 0x05

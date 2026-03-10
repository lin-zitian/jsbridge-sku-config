# 加湿器(Humidifier)配置指南

## 设备类型
- **类型标识**: `devices.types.humidifier`
- **适用设备**: 加湿器、香薰机、雾化器

## 参考 SKU
- **H7140**: 标准加湿器，支持工作模式、湿度调节、夜灯功能

## 常用 Instance 能力

### 1. 基础控制
- `powerSwitch`: 电源开关
- `workMode`: 工作模式(手动/自定义/自动)
- `humidity`: 湿度调节(40-80%)

### 2. 开关类
- `nightlightToggle`: 夜灯开关
- `mistToggle`: 出雾开关
- `warmMistToggle`: 热雾开关

### 3. 灯光控制
- `brightness`: 夜灯亮度(1-100)
- `colorRgb`: 夜灯颜色
- `nightlightScene`: 夜灯场景(Forest/Ocean/Wetland/Leisurely/Sleep)

### 4. 延时功能
- `mistDuration`: 出雾延时关
- `nightlightDuration`: 夜灯延时关

### 5. 事件告警
- `lackWaterEvent`: 缺水事件

## BLE 指令配置

### 一致性指令(所有加湿器通用)

#### powerSwitch - 电源开关
```javascript
action: {
  write: [{ bleDefine: ['33', '01'], valIndex: [2] }],
  read: ['aa', '01']
},
payloadDefine: { cmdType: 'multiSync' }
```

#### nightlightToggle - 夜灯开关
```javascript
action: {
  write: [{ bleDefine: ['3a', '1b', '01', '01', '01'], valIndex: [4] }]
},
payloadDefine: { cmdType: 'multiSync' }
```

#### brightness - 夜灯亮度
```javascript
action: {
  write: [{ bleDefine: ['3a', '1b', '01', '02', '32'], valIndex: [4] }]
},
payloadDefine: { cmdType: 'multiSync' }
```

#### colorRgb - 夜灯颜色
```javascript
action: {
  write: [{ bleDefine: ['3a', '1b', '05', '0d', '32', '32', '32'], valIndex: [4, 5, 6] }]
},
payloadDefine: { cmdType: 'multiSync' }
```

### 可能变化的指令

#### workMode - 工作模式
**H7140 结构**:
```javascript
action: {
  write: [{ bleDefine: ['3a', '05', '01'], valIndex: [2, 3] }],
  read: [{ bleDefine: ['aa', '05'] }, { bleDefine: ['aa', '05', '01'] }]
},
parameters: {
  dataType: 'STRUCT',
  fields: [
    {
      fieldName: 'workMode',
      options: [
        { name: 'Manual', value: 1 },
        { name: 'Custom', value: 2 },
        { name: 'Auto', value: 3 }
      ]
    },
    {
      fieldName: 'modeValue',
      options: [
        { name: 'Manual', options: [{ value: 1 }, { value: 2 }, ...{ value: 9 }] },
        { name: 'Custom', defaultValue: 0 },
        { name: 'Auto', defaultValue: 0 }
      ]
    }
  ]
}
```

**注意**: 不同型号的加湿器可能有不同的工作模式选项和档位数量

#### humidity - 湿度调节
**H7140 结构**:
```javascript
action: {
  write: [{ bleDefine: ['3a', '05', '03', '01'], valIndex: [3] }]
},
parameters: {
  dataType: 'INTEGER',
  unit: 'unit.percent',
  range: { min: 40, max: 80, precision: 1 }
}
```

**注意**: 湿度范围可能因型号而异(常见: 40-80%, 30-90%)

#### nightlightScene - 夜灯场景
**H7140 场景**:
- Forest(森林) - 1
- Ocean(海洋) - 2
- Wetland(湿地) - 3
- Leisurely(悠闲) - 4
- Sleep(睡眠) - 5

**注意**: 不同型号可能有不同的场景选项

## 配置流程

### Step 1: 确认基础信息
询问用户:
1. SKU 编号
2. goodsType(产品类型)
3. 是否支持夜灯功能?
4. 是否支持出雾控制?

### Step 2: 确认工作模式
询问用户:
1. 支持哪些工作模式? (Manual/Custom/Auto/Sleep等)
2. Manual 模式有几个档位? (通常1-9档)
3. 是否支持自动模式?

### Step 3: 确认湿度控制
询问用户:
1. 湿度调节范围? (默认40-80%)
2. 精度? (默认1%)

### Step 4: 确认夜灯功能
如果支持夜灯,询问用户:
1. 是否支持亮度调节? (1-100)
2. 是否支持颜色调节? (RGB)
3. 支持哪些场景模式?

### Step 5: 确认事件告警
询问用户:
1. 是否支持缺水检测?
2. 是否有其他告警事件?

### Step 6: BLE 指令配置
- 基础指令(powerSwitch/nightlightToggle/brightness/colorRgb)使用标准配置
- workMode/humidity/nightlightScene 需要根据用户提供的信息配置

## 常见配置模式

### 模式 1: 基础加湿器
- powerSwitch
- workMode (Manual/Auto)
- humidity
- lackWaterEvent

### 模式 2: 带夜灯加湿器
- 基础加湿器 +
- nightlightToggle
- brightness
- colorRgb
- nightlightScene

### 模式 3: 高级加湿器
- 带夜灯加湿器 +
- mistToggle
- warmMistToggle
- mistDuration
- nightlightDuration

## 注意事项

1. **cmdType**: 加湿器通常使用 `multiSync`
2. **工作模式**: Manual 模式的档位数量需要确认
3. **夜灯指令**: 夜灯相关指令(0x1b)通常一致
4. **湿度范围**: 不同型号范围可能不同,需要确认
5. **场景名称**: 场景名称和数量可能因型号而异

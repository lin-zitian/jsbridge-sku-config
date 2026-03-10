# action 配置的作用说明

本文档详细解释 SKU 配置中 `action` 字段的作用，特别是音乐模式和其他能力的区别。

---

## action 的核心作用

`action` 定义了如何将用户的控制数据编码成 BLE 蓝牙指令。

### 基本结构

```javascript
action: {
  write: [
    {
      bleDefine: ['33', '05', '15', '01', 'ff', 'ff', 'ff'],  // BLE 指令模板
      valIndex: [4, 5, 6]  // 数据填充位置
    }
  ]
}
```

- **bleDefine**: BLE 指令的十六进制模板
- **valIndex**: 指定用户数据应该填充到模板的哪些索引位置

---

## 不同 cmdType 的 action 作用

### 1. 固定指令类型（无需 action）

这些能力使用标准的 cmdType，不需要配置 action：

#### powerSwitch (cmdType: 'turn')
```javascript
{
  instance: InstanceEnum.powerSwitch.value,
  type: 'devices.capabilities.on_off',
  payloadDefine: {
    cmdType: 'turn'  // 标准开关指令
  },
  // ❌ 不需要 action
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

#### brightness (cmdType: 'brightness')
```javascript
{
  instance: InstanceEnum.brightness.value,
  type: 'devices.capabilities.range',
  payloadDefine: {
    cmdType: 'brightness'  // 标准亮度指令
  },
  // ❌ 不需要 action
  parameters: {
    dataType: 'INTEGER',
    unit: 'unit.percent',
    range: { min: 1, max: 100, precision: 1 }
  }
}
```

#### colorRgb / colorTemperatureK (cmdType: 'colorwc')
```javascript
{
  instance: InstanceEnum.colorRgb.value,
  type: 'devices.capabilities.color_setting',
  payloadDefine: {
    cmdType: 'colorwc'  // 标准颜色指令
  },
  // ❌ 不需要 action
  parameters: {
    dataType: 'INTEGER',
    range: { min: 0, max: 0xffffff, precision: 1 }
  }
}
```

---

### 2. 自定义指令类型（需要 action）

这些能力使用 `cmdType: 'ptReal'` 或 `'multiSync'`，必须配置 action。

#### 分段颜色控制 (segmentedColorRgb)

**用户输入**：
```javascript
{
  segment: [0, 1, 2],  // 要控制的段
  rgb: 0xff0000        // 红色
}
```

**action 配置**：
```javascript
{
  instance: InstanceEnum.segmentedColorRgb.value,
  type: 'devices.capabilities.segment_color_setting',
  action: {
    write: [
      {
        bleDefine: ['33', '05', '15', '01', 'ff', 'ff', 'ff', '00', '00', '00', '00', '00', 'ff', 'ff'],
        valIndex: [4, 5, 6, 12, 13]
        //         ↑  ↑  ↑   ↑   ↑
        //         R  G  B  seg seg
      }
    ]
  },
  payloadDefine: {
    cmdType: 'ptReal'
  }
}
```

**处理流程**：
1. `segmentColorAnalysis` 函数被调用
2. 从用户数据中提取 RGB 值：`[0xff, 0x00, 0x00]`
3. 从用户数据中提取 segment 值：`[0, 1, 2]`
4. 使用 `bleCommandPadding` 将数据填充到 bleDefine 模板的 valIndex 位置
5. 生成最终的 BLE 指令

---

### 3. 音乐模式的特殊情况

音乐模式比较特殊，它有**两套指令系统**：

#### 系统 1: 动态参数指令（使用 action）

用于控制音乐模式的**动态参数**（灵敏度、自动颜色、RGB）：

**用户输入**：
```javascript
{
  musicMode: 0,        // 选择第 0 个音乐模式
  sensitivity: 50,     // 灵敏度 50%
  autoColor: 1,        // 自动颜色开启
  rgb: 0xff0000        // 红色
}
```

**action 配置**：
```javascript
{
  instance: InstanceEnum.musicMode.value,
  type: 'devices.capabilities.music_setting',
  action: {
    write: [
      {
        bleDefine: ['33', '05', '13', '05', '32', '00', '01', 'ff', 'ff', 'ff'],
        valIndex: [3, 4, 5, 6, 7, 8, 9]
        //         ↑  ↑  ↑  ↑  R  G  B
        //      mode sen sub auto
      }
    ]
  },
  payloadDefine: {
    cmdType: 'ptReal'
  }
}
```

**处理流程**：
1. `musicModeAnalysis` 函数被调用
2. 从用户数据中提取：musicMode=0, sensitivity=50, autoColor=1, rgb=[0xff, 0x00, 0x00]
3. 使用 `bleCommandPadding` 将数据填充到 bleDefine 模板的 valIndex 位置
4. 生成包含动态参数的 BLE 指令

#### 系统 2: 预定义模式指令（使用 MusicMode 常量）

用于选择**具体的音乐模式**（Ripple、Gridding 等）：

**MusicMode 常量**：
```javascript
const MusicMode = [
  {
    name: modeLanguageConst.Ripple,
    ctrData: [
      "owABAkFFB/8AAP9/AP//AAD/ACM=",
      "o/8AAP8A//+LAP8AAP8KAAAAACI=",
      "MwUTRWMAAAAAAAAAAAAAAAAAAAM="
    ]
  },
  {
    name: modeLanguageConst.Gridding,
    ctrData: [
      "owABAkFEB/8AAP9/AP//AAD/ACI=",
      "o/8AAP8A//+LAP8AAP8KAAAAACI=",
      "MwUTRGMAAAAAAAAAAAAAAAAAAAI="
    ]
  }
  // ... 更多音乐模式
];
```

**处理流程**：
1. `getMqttPayload` 函数调用 `musicModeMqttPayloadInterceptor`
2. 根据 `input.data.musicMode` 索引，从 MusicMode 数组中获取对应的 ctrData
3. **直接替换整个 command**：`output.msg.data.command = MusicMode[input.data.musicMode].ctrData`
4. 这些 ctrData 是完整的预定义 BLE 指令序列

**关键代码**：
```javascript
function musicModeMqttPayloadInterceptor(input, capaInstance, output) {
  if (equalsIgnoreCase(input.instance, InstanceEnum.musicMode.value)) {
    // 直接使用 MusicMode 常量中的完整指令
    output.msg.data.command = MusicMode[input.data.musicMode].ctrData;
    return output;
  }
  return output;
}
```

---

## 为什么音乐模式需要 action？

虽然 MusicMode 常量已经定义了完整的模式指令，但 action 仍然是必需的，原因如下：

### 1. 动态参数控制

action 用于处理音乐模式的**动态参数**：
- **sensitivity**（灵敏度）：用户可以调整 0-100%
- **autoColor**（自动颜色）：开启/关闭
- **rgb**（颜色）：用户可以选择任意颜色

这些参数不在 MusicMode 常量中，需要通过 action 动态编码。

### 2. 两种控制场景

**场景 A：选择音乐模式**
```javascript
// 用户选择 "Ripple" 模式
{
  musicMode: 0,  // 索引 0 = Ripple
  sensitivity: 50,
  autoColor: 1,
  rgb: 0xff0000
}
// 使用 MusicMode[0].ctrData 的完整指令
```

**场景 B：调整当前模式的参数**
```javascript
// 用户调整灵敏度和颜色，但不改变模式
{
  musicMode: 0,  // 仍然是 Ripple
  sensitivity: 80,  // 调整灵敏度
  autoColor: 0,     // 关闭自动颜色
  rgb: 0x00ff00     // 改为绿色
}
// 使用 action 生成包含新参数的指令
```

### 3. 系统架构需要

`musicModeAnalysis` 函数需要 action 来生成 BLE 指令：

```javascript
function musicModeAnalysis(input = {}, capaInstance = {}) {
  const { data, action } = input;
  let { musicMode, sensitivity, autoColor, rgb } = data;
  
  // 从 capaInstance.action[action] 获取 BLE 模板
  const cmdArray = capaInstance.action[action];
  const bleValue = [musicMode, sensitivity, autoColor, ...rgb];
  
  // 使用模板和数据生成 BLE 指令
  const bleCommands = bleCommandPadding(action, bleValue, cmdArray);
  
  return { ble: bleCommands };
}
```

如果没有 action，这个函数无法工作。

---

## 其他需要 action 的能力

### gradientToggle（渐变开关）

```javascript
{
  instance: InstanceEnum.gradientToggle.value,
  type: 'devices.capabilities.toggle',
  action: {
    write: [
      {
        bleDefine: ['33', 'a3', '01'],
        valIndex: [2]  // 开关值填充到索引 2
      }
    ]
  },
  payloadDefine: {
    cmdType: 'ptReal'
  }
}
```

### mainLightToggle（主灯开关）

```javascript
{
  instance: InstanceEnum.mainLightToggle.value,
  type: 'devices.capabilities.toggle',
  action: {
    write: [
      {
        bleDefine: ['33', '36', '00'],
        valIndex: [3]  // 开关值填充到索引 3
      }
    ]
  },
  payloadDefine: {
    cmdType: 'ptReal'
  }
}
```

### workMode（工作模式）

```javascript
{
  instance: 'workMode',
  type: 'devices.capabilities.work_mode',
  action: {
    write: [
      {
        bleDefine: ['3a', '05', '01'],
        valIndex: [2, 3]  // 模式和档位值
      }
    ]
  },
  payloadDefine: {
    cmdType: 'multiSync'
  }
}
```

---

## 总结

### 固定指令能力（7 个，无需 action）

1. **powerSwitch** - cmdType: 'turn'
2. **brightness** - cmdType: 'brightness'
3. **colorRgb** - cmdType: 'colorwc'
4. **colorTemperatureK** - cmdType: 'colorwc'
5. **lightScene** - 外部填充
6. **diyScene** - 外部填充
7. **snapshot** - 外部填充

### 自定义指令能力（需要 action）

所有使用 `cmdType: 'ptReal'` 或 `'multiSync'` 的能力都需要 action：

- **segmentedColorRgb** / **segmentedBrightness** - 分段控制
- **musicMode** - 音乐模式（动态参数）
- **gradientToggle** - 渐变开关
- **mainLightToggle** / **backgroundLightToggle** - 灯光开关
- **workMode** - 工作模式
- **humidity** - 湿度控制
- **nightlightToggle** - 夜灯开关
- **nightlightScene** - 夜灯场景
- 其他自定义能力

### 音乐模式的双重机制

1. **action**：用于动态参数（sensitivity, autoColor, rgb）
2. **MusicMode 常量**：用于预定义的音乐模式指令（Ripple, Gridding 等）

两者配合使用，缺一不可。

---

## 配置建议

### 1. 判断是否需要 action

查看 `payloadDefine.cmdType`：
- 如果是 `'turn'`, `'brightness'`, `'colorwc'` → 不需要 action
- 如果是 `'ptReal'`, `'multiSync'` → 需要 action

### 2. 配置 action

如果需要 action，必须提供：
- **bleDefine**：BLE 指令模板（十六进制数组）
- **valIndex**：数据填充位置（数字数组）

### 3. 如果暂时无法提供

使用 TODO 模板：
```javascript
action: {
  // TODO: 请提供 BLE 指令配置
  // write: [
  //   {
  //     bleDefine: ['33', '05', '...'],
  //     valIndex: [2, 3, ...]
  //   }
  // ]
}
```

---

## 参考文档

- `fixed-ble-instructions.md` - 固定指令能力列表
- `ble-instruction-guide.md` - BLE 指令配置详细指南
- `H1232-template.md` - 完整实现示例

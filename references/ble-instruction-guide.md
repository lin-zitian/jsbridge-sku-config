# BLE 指令配置指南

本文档说明哪些 instance 需要配置自定义 BLE 指令，以及如何配置。

**完整的固定指令列表**: 请查看 `#[[file:references/fixed-ble-instructions.md]]`

## 核心规则

### 规则 1: 固定指令能力（无需 action）
以下 7 个 instance 使用固定 cmdType，**不需要配置 action**：
1. powerSwitch (turn)
2. brightness (brightness)
3. colorRgb (colorwc/color)
4. colorTemperatureK (colorwc)
5. lightScene (外部填充)
6. diyScene (外部填充)
7. snapshot (外部填充)

### 规则 2: 其他所有能力都需要 action
**所有不在上述列表中的 instance 都需要配置 action**。

如果用户暂时无法提供：
```javascript
action: {
  // TODO: 请提供 BLE 指令配置
  // write: [{ bleDefine: ['33', '...'], valIndex: [2, 3] }]
}
```

## 指令配置分类

### 特殊设备类型说明

**制冰机（devices.types.ice_maker）**：
- 制冰机的 BLE 指令差异较大，即使是相同的 instance，不同 SKU 的指令也可能完全不同
- 详细配置指南请参考：`ice-maker-guide.md`
- 核心差异 instance：`powerSwitch`, `iceMakingToggle`, `workMode`, `nightlightScene`

---

### ✅ 无需配置 BLE 指令的 Instance

以下 instance 使用标准 cmdType，**无需询问用户配置 BLE 指令**：

| Instance | cmdType | 说明 |
|----------|---------|------|
| `powerSwitch` | `turn` | 电源开关，使用标准开关指令 |
| `brightness` | `brightness` | 整体亮度，使用标准亮度指令 |
| `colorRgb` | `colorwc` | RGB 颜色，使用标准颜色指令 |
| `colorTemperatureK` | `colorwc` | 色温控制，使用标准颜色指令 |
| `lightScene` | - | 灯光场景，type 为 dynamic_scene |
| `diyScene` | - | DIY 场景，type 为 dynamic_scene |
| `snapshot` | `ptReal` | 快照，通常无需自定义 |

**配置示例**（无需 action）：

```javascript
{
  instance: InstanceEnum.powerSwitch.value,
  type: 'devices.capabilities.on_off',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'turn'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

---

### ⚠️ 需要配置 BLE 指令的 Instance

以下 instance 使用 `ptReal` cmdType，**必须询问用户提供 BLE 指令**。

#### 1. segmentedColorRgb (分段颜色控制)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引
- `parameters.fields[0].elementRange.max`: 最大段号
- `parameters.fields[0].size.max`: 最大段数

**参考示例**（H1232 - 13段）：
```javascript
{
  instance: InstanceEnum.segmentedColorRgb.value,
  type: 'devices.capabilities.segment_color_setting',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['33', '05', '15', '01', 'ff', 'ff', 'ff', '00', '00', '00', '00', '00', 'ff', 'ff'],
        valIndex: [4, 5, 6, 12, 13]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'segment',
        dataType: 'Array',
        required: true,
        elementType: 'INTEGER',
        elementRange: { max: 12, min: 0 },  // 0-12 共13段
        size: { max: 13, min: 1 }
      },
      {
        fieldName: 'rgb',
        dataType: 'INTEGER',
        required: true,
        range: { min: 0, max: 0xffffff, precision: 1 }
      }
    ]
  }
}
```

**询问用户**：
1. 设备支持多少段颜色控制？（如：13段，segment: 0-12）
2. BLE 指令是什么？（bleDefine 数组）
3. 数据位置索引是什么？（valIndex 数组）

---

#### 2. segmentedBrightness (分段亮度控制)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引
- `parameters.fields[0].elementRange.max`: 最大段号
- `parameters.fields[0].size.max`: 最大段数

**参考示例**（H1232 - 13段）：
```javascript
{
  instance: InstanceEnum.segmentedBrightness.value,
  type: 'devices.capabilities.segment_color_setting',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['33', '05', '15', '02', '32', '00', '00', '00', '00', '00'],
        valIndex: [4, 5, 6, 7, 8, 9]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'segment',
        dataType: 'Array',
        required: true,
        elementType: 'INTEGER',
        elementRange: { max: 12, min: 0 },
        size: { max: 13, min: 1 }
      },
      {
        fieldName: 'brightness',
        dataType: 'INTEGER',
        required: true,
        range: { min: 0, max: 100, precision: 1 }
      }
    ]
  }
}
```

**询问用户**：
1. 设备支持多少段亮度控制？（如：13段，segment: 0-12）
2. BLE 指令是什么？（bleDefine 数组）
3. 数据位置索引是什么？（valIndex 数组）

---

#### 3. musicMode (音乐模式)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引
- `parameters.fields[0].options`: 音乐模式列表（后期配置）

**参考示例**（H1232）：
```javascript
{
  instance: InstanceEnum.musicMode.value,
  type: 'devices.capabilities.music_setting',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['33', '05', '13', '05', '32', '00', '01', 'ff', 'ff', 'ff'],
        valIndex: [3, 4, 5, 6, 7, 8, 9]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'musicMode',
        dataType: 'ENUM',
        required: true,
        options: []  // TODO: 后续配置
      },
      {
        fieldName: 'sensitivity',
        dataType: 'INTEGER',
        required: true,
        unit: 'unit.percent',
        range: { min: 0, max: 100, precision: 1 }
      },
      {
        fieldName: 'autoColor',
        dataType: 'ENUM',
        required: false,
        options: [
          { name: 'on', value: 1 },
          { name: 'off', value: 0 }
        ]
      },
      {
        fieldName: 'rgb',
        dataType: 'INTEGER',
        required: false,
        range: { min: 0, max: 0xffffff, precision: 1 }
      }
    ]
  }
}
```

**询问用户**：
1. BLE 指令是什么？（bleDefine 数组）
2. 数据位置索引是什么？（valIndex 数组）
3. 音乐模式数据暂时不提供，添加 TODO 标记

---

#### 4. gradientToggle (渐变开关)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引

**参考示例**（H1232）：
```javascript
{
  instance: InstanceEnum.gradientToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [{ bleDefine: ['33', 'a3', '01'], valIndex: [2] }],
    read: []
  },
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

**询问用户**：
1. BLE 指令是什么？（bleDefine 数组）
2. 数据位置索引是什么？（valIndex 数组）

---

#### 5. mainLightToggle (主灯开关)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引

**参考示例**（H1232）：
```javascript
{
  instance: InstanceEnum.mainLightToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [{ bleDefine: ['33', '30', '00'], valIndex: [3] }],
    read: []
  },
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

**询问用户**：
1. BLE 指令是什么？（bleDefine 数组）
2. 数据位置索引是什么？（valIndex 数组）

---

#### 6. backgroundLightToggle (背灯开关)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引

**参考示例**（H1232）：
```javascript
{
  instance: InstanceEnum.backgroundLightToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [{ bleDefine: ['33', '30', '01'], valIndex: [3] }],
    read: []
  },
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

**询问用户**：
1. BLE 指令是什么？（bleDefine 数组）
2. 数据位置索引是什么？（valIndex 数组）

---

#### 7. leftLightToggle / rightLightToggle (左右灯开关)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引

**参考示例**（H6046）：
```javascript
// 左灯
{
  instance: InstanceEnum.leftLightToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'
  },
  action: {
    write: [{ bleDefine: ['33', '36', '01', 'ff'], valIndex: [2] }]
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}

// 右灯
{
  instance: InstanceEnum.rightLightToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'
  },
  action: {
    write: [{ bleDefine: ['33', '36', 'ff', '01'], valIndex: [3] }]
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

**询问用户**：
1. BLE 指令是什么？（bleDefine 数组）
2. 数据位置索引是什么？（valIndex 数组）

---

#### 8. nightlightToggle (夜灯开关)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引

**参考示例**（H7140）：
```javascript
{
  instance: InstanceEnum.nightlightToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '1b', '01', '01', '01'],
        valIndex: [4]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

**询问用户**：
1. BLE 指令是什么？（bleDefine 数组）
2. 数据位置索引是什么？（valIndex 数组）
3. cmdType 是什么？（ptReal 或 multiSync）

---

#### 9. workMode (工作模式)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引
- `parameters.fields[0].options`: 模式列表

**参考示例**（H7140）：
```javascript
{
  instance: 'workMode',
  type: 'devices.capabilities.work_mode',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '05', '01'],
        valIndex: [2, 3]
      }
    ],
    read: [{ bleDefine: ['aa', '05'] }, { bleDefine: ['aa', '05', '01'] }]
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'workMode',
        dataType: 'ENUM',
        required: true,
        options: [
          { name: 'Manual', value: 1 },
          { name: 'Custom', value: 2 },
          { name: 'Auto', value: 3 }
        ]
      },
      {
        fieldName: 'modeValue',
        dataType: 'ENUM',
        required: false,
        options: [
          {
            name: 'Manual',
            options: [
              { value: 1 },
              { value: 2 },
              { value: 3 }
            ]
          },
          { name: 'Custom', defaultValue: 0 },
          { name: 'Auto', defaultValue: 0 }
        ]
      }
    ]
  }
}
```

**询问用户**：
1. BLE 指令是什么？（bleDefine 数组）
2. 数据位置索引是什么？（valIndex 数组）
3. 支持哪些工作模式？（options 列表）
4. cmdType 是什么？（ptReal 或 multiSync）

---

#### 10. humidity (湿度控制)

**必须配置**：
- `action.write[0].bleDefine`: BLE 指令字节数组
- `action.write[0].valIndex`: 数据位置索引
- `parameters.range`: 湿度范围

**参考示例**（H7140）：
```javascript
{
  instance: InstanceEnum.humidity.value,
  type: 'devices.capabilities.range',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '05', '03', '01'],
        valIndex: [3]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'INTEGER',
    unit: 'unit.percent',
    range: {
      min: 40,
      max: 80,
      precision: 1
    }
  }
}
```

**询问用户**：
1. BLE 指令是什么？（bleDefine 数组）
2. 数据位置索引是什么？（valIndex 数组）
3. 湿度范围是多少？（min/max）
4. cmdType 是什么？（ptReal 或 multiSync）

---

## BLE 指令配置流程

### 步骤 1: 识别需要配置的 Instance

根据用户提供的 instance 列表，识别哪些需要配置 BLE 指令：

```javascript
const needsBleConfig = [
  'segmentedColorRgb',
  'segmentedBrightness',
  'musicMode',
  'gradientToggle',
  'mainLightToggle',
  'backgroundLightToggle',
  'leftLightToggle',
  'rightLightToggle',
  'nightlightToggle',
  'workMode',
  'humidity'
];
```

### 步骤 2: 询问用户配置

对于每个需要配置的 instance，询问用户：

1. **BLE 指令** (bleDefine)
   - 格式：字符串数组，如 `['33', '05', '15', '01', 'ff']`
   - 说明：BLE 指令的十六进制字节数组

2. **数据位置索引** (valIndex)
   - 格式：数字数组，如 `[4, 5, 6]`
   - 说明：数据值在 BLE 指令中的位置（从0开始）

3. **特殊参数**（根据 instance 类型）
   - 分段控制：段数
   - 工作模式：模式列表
   - 湿度控制：范围
   - 音乐模式：暂时不提供，添加 TODO

### 步骤 3: 提供参考值

如果用户暂时无法提供 BLE 指令，可以：

1. 使用参考 SKU 的指令（如 H1232）
2. 在任务清单中标记需要确认
3. 提醒用户后续需要更新

### 步骤 4: 验证配置

生成配置后，检查：

1. bleDefine 是否为有效的十六进制字符串数组
2. valIndex 是否在 bleDefine 的有效范围内
3. 特殊参数是否合理（如段数、范围等）

---

## 常见问题

### Q: 如何判断一个 instance 是否需要配置 BLE 指令？

A: 查看该 instance 是否在"需要配置 BLE 指令的 Instance"列表中。通常使用 `ptReal` 或 `multiSync` cmdType 的 instance 需要配置。

### Q: 用户不知道 BLE 指令怎么办？

A: 可以先使用参考 SKU（如 H1232）的指令，并在任务清单中标记需要确认。提醒用户后续需要根据实际设备更新。

### Q: valIndex 是什么意思？

A: valIndex 指定数据值在 BLE 指令中的位置。例如 `valIndex: [4, 5, 6]` 表示将数据的3个字节分别填充到指令的第4、5、6位置（从0开始计数）。

### Q: 不同 SKU 的同一个 instance 指令会不同吗？

A: 会的。即使是同一个 instance（如 gradientToggle），不同 SKU 的 BLE 指令可能完全不同，需要根据实际设备配置。

### Q: 音乐模式为什么要分两次配置？

A: 音乐模式的 BLE 指令可以先配置，但音乐模式的具体数据（MusicMode 数组）通常后期才提供，所以先添加 TODO 标记，后续再配置。

---

## 参考资料

- H1232 完整示例：`references/H1232-template.md`
- H7140 加湿器示例：`jsbridge-be/packages/H7140/src/index.mjs`
- H6046 灯具示例：`jsbridge-be/packages/H6046/src/index.mjs`
- 标准模板：`references/standard-sku-template.md`

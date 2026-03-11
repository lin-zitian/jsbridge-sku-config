# 外部填充数据字段说明

本文档说明哪些字段的数据由外部系统填充，在配置 SKU 时**禁止编造数据**。

## 外部填充字段列表

## 外部填充字段列表

### 快速对比表

| 字段 | dataType | options | payloadDefine | 说明 |
|------|----------|---------|---------------|------|
| **lightScene** | `ENUM` | `[]` | ❌ 不需要 | 灯光场景 |
| **diyScene** | `ENUM` | `[]` | ❌ 不需要 | DIY 场景 |
| **snapshot** | `ENUM` | `[]` | ✅ 需要 `{ cmdType: 'ptReal' }` | 快照 |
| **musicMode 字段** | `ENUM` | `[]` | - | 音乐模式选项 |

**关键规则**:
- 所有外部填充字段都使用 `dataType: 'ENUM'`
- 所有外部填充字段的 `options` 都是空数组 `[]`
- **只有 snapshot 需要 `payloadDefine`**
- lightScene 和 diyScene 不要添加 `payloadDefine`

---

### 1. lightScene (灯光场景)

**字段位置**: `parameters.options`

**正确配置**:
```javascript
{
  instance: InstanceEnum.lightScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  parameters: {
    dataType: 'ENUM',
    options: []  // ✅ 必须为空数组，由外部系统填充
  }
}
```

**错误配置 1 - 编造 options 数据**:
```javascript
{
  parameters: {
    dataType: 'ENUM',
    options: [  // ❌ 禁止编造场景数据
      { name: 'Rainbow', value: 1 },
      { name: 'Sunset', value: 2 }
    ]
  }
}
```

**错误配置 2 - 添加 payloadDefine**:
```javascript
{
  instance: InstanceEnum.lightScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'  // ❌ lightScene 不需要 payloadDefine
  },
  parameters: {
    dataType: 'ENUM',
    options: []
  }
}
```

**关键点**:
- ✅ 必须使用 `dataType: 'ENUM'`
- ✅ `options` 必须为空数组 `[]`
- ❌ 不要添加 `payloadDefine`
- ❌ 不要编造 options 数据

**说明**: 灯光场景数据由外部场景管理系统提供，不同设备的场景可能不同。

---

### 2. diyScene (DIY 场景)

**字段位置**: `parameters.options`

**正确配置**:
```javascript
{
  instance: InstanceEnum.diyScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  parameters: {
    dataType: 'ENUM',  // ✅ 必须是 ENUM
    options: []  // ✅ 必须为空数组，由外部系统填充
  }
}
```

**错误配置 1 - 编造 options 数据**:
```javascript
{
  parameters: {
    dataType: 'ENUM',
    options: [  // ❌ 禁止编造 DIY 场景数据
      { name: 'Custom1', value: 1 },
      { name: 'Custom2', value: 2 }
    ]
  }
}
```

**错误配置 2 - 使用 STRUCT 结构**:
```javascript
{
  instance: InstanceEnum.diyScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'  // ❌ 不需要 payloadDefine
  },
  parameters: {
    dataType: 'STRUCT',  // ❌ 错误！应该是 ENUM
    fields: [  // ❌ 不需要 fields
      {
        fieldName: 'sceneId',
        dataType: 'INTEGER',
        required: true
      },
      {
        fieldName: 'sceneData',
        dataType: 'STRING',
        required: true
      }
    ]
  }
}
```

**关键点**:
- ✅ 必须使用 `dataType: 'ENUM'`
- ✅ `options` 必须为空数组 `[]`
- ❌ 不要添加 `payloadDefine`
- ❌ 不要使用 `dataType: 'STRUCT'`
- ❌ 不要添加 `fields` 字段

**说明**: DIY 场景由用户自定义，数据格式由外部系统管理。与 lightScene 和 snapshot 一样，使用简单的 ENUM 结构，options 为空数组。

---

### 3. snapshot (快照)

**字段位置**: `parameters.options`

**正确配置**:
```javascript
{
  instance: InstanceEnum.snapshot.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'  // ✅ snapshot 需要 payloadDefine
  },
  parameters: {
    dataType: 'ENUM',
    options: []  // ✅ 必须为空数组，由外部系统填充
  }
}
```

**关键点**:
- ✅ 必须使用 `dataType: 'ENUM'`
- ✅ `options` 必须为空数组 `[]`
- ✅ **snapshot 需要 `payloadDefine: { cmdType: 'ptReal' }`**（与 lightScene、diyScene 不同）
- ❌ 不要编造 options 数据

**说明**: 快照功能保存当前设备状态，数据由外部系统管理。snapshot 是唯一需要 payloadDefine 的外部填充字段。

---

### 4. musicMode 的 musicMode 字段

**字段位置**: `parameters.fields[0].options` (musicMode 字段)

**正确配置**:
```javascript
{
  instance: InstanceEnum.musicMode.value,
  type: 'devices.capabilities.music_setting',
  ctrlPlatformSupported: ['openApi'],
  action: {  // ✅ 必须包含完整的 action 配置
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
        options: []  // ✅ 必须为空数组，后续填充
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

**错误配置**:
```javascript
{
  parameters: {
    fields: [
      {
        fieldName: 'musicMode',
        options: [  // ❌ 禁止编造音乐模式数据
          { name: 'Ripple', value: 0 },
          { name: 'Beat', value: 1 }
        ]
      }
    ]
  }
}
```

**或者缺少 action**:
```javascript
{
  instance: InstanceEnum.musicMode.value,
  // ❌ 缺少 action 配置
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    // ...
  }
}
```

**说明**: 
- musicMode 字段的 options 后续配置时填充
- **即使 options 为空，也必须包含完整的 action 配置**
- 音乐模式数据使用 `modeLanguageConst` 枚举类
- 参考 H1232 的 MusicMode 实现

---

## 配置检查清单

在配置 SKU 时，检查以下项目：

- [ ] lightScene 的 options 是否为空数组 `[]`
- [ ] lightScene 是否没有添加 payloadDefine（不需要）
- [ ] diyScene 的 options 是否为空数组 `[]`
- [ ] diyScene 是否使用 ENUM 而不是 STRUCT
- [ ] diyScene 是否没有添加 payloadDefine（不需要）
- [ ] snapshot 的 options 是否为空数组 `[]`
- [ ] snapshot 是否包含 `payloadDefine: { cmdType: 'ptReal' }`（需要）
- [ ] musicMode 的 musicMode 字段 options 是否为空数组 `[]`
- [ ] musicMode 是否包含完整的 action 配置
- [ ] 没有编造上述字段的数据

---

## 为什么不能编造数据？

1. **数据一致性**: 这些数据由外部系统统一管理，确保所有设备使用相同的数据格式
2. **动态更新**: 场景和音乐模式可能会动态更新，硬编码会导致无法更新
3. **多语言支持**: 这些数据包含多语言信息，由外部系统提供
4. **版本管理**: 外部系统负责数据的版本管理和兼容性

---

## 后续配置流程

### lightScene / diyScene / snapshot
这些字段的数据完全由外部系统管理，SKU 配置中保持 options 为空数组即可。

### musicMode
1. **初次配置**: 
   - musicMode 字段 options 为空数组
   - 必须配置完整的 action (BLE 指令)
   - 在代码中创建空的 MusicMode 数组，添加 TODO 注释

2. **后续配置**:
   - 用户提供音乐模式数据后
   - 唤醒 skill: "配置 [SKU] 的音乐模式"
   - 填充 MusicMode 数组
   - 实现 musicCapabilitiesInterceptor 函数
   - 实现 musicModeMqttPayloadInterceptor 函数
   - 导出 getMusicMode 函数

---

## 参考文档

- H1232 完整实现: `references/H1232-template.md`
- 灯光设备配置指南: `references/light-guide.md`
- BLE 指令配置指南: `references/ble-instruction-guide.md`
- 音乐模式配置: SKILL.md 中的"音乐模式配置规则"章节

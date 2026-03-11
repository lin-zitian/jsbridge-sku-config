# 外部填充字段快速参考

本文档提供外部填充字段的快速配置参考。

---

## 配置对比表

| 字段 | dataType | options | payloadDefine | 说明 |
|------|----------|---------|---------------|------|
| **lightScene** | `ENUM` | `[]` | ❌ 不需要 | 灯光场景，外部填充 |
| **diyScene** | `ENUM` | `[]` | ❌ 不需要 | DIY 场景，外部填充 |
| **snapshot** | `ENUM` | `[]` | ✅ `{ cmdType: 'ptReal' }` | 快照，外部填充 |
| **musicMode 字段** | `ENUM` | `[]` | - | 音乐模式选项，后续填充 |

---

## 固定配置模板

### lightScene（灯光场景）

```javascript
{
  instance: InstanceEnum.lightScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  parameters: {
    dataType: 'ENUM',
    options: []
  }
}
```

**关键点**:
- ✅ 使用 `dataType: 'ENUM'`
- ✅ `options` 为空数组 `[]`
- ❌ 不要添加 `payloadDefine`
- ❌ 不要编造 options 数据

---

### diyScene（DIY 场景）

```javascript
{
  instance: InstanceEnum.diyScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  parameters: {
    dataType: 'ENUM',
    options: []
  }
}
```

**关键点**:
- ✅ 使用 `dataType: 'ENUM'`（不是 STRUCT）
- ✅ `options` 为空数组 `[]`
- ❌ 不要添加 `payloadDefine`
- ❌ 不要添加 `fields` 字段
- ❌ 不要编造 options 数据

---

### snapshot（快照）

```javascript
{
  instance: InstanceEnum.snapshot.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'  // ✅ snapshot 需要这个
  },
  parameters: {
    dataType: 'ENUM',
    options: []
  }
}
```

**关键点**:
- ✅ 使用 `dataType: 'ENUM'`
- ✅ `options` 为空数组 `[]`
- ✅ **必须添加** `payloadDefine: { cmdType: 'ptReal' }`
- ❌ 不要编造 options 数据

---

## 常见错误

### ❌ 错误 1: lightScene 添加了 payloadDefine

```javascript
// ❌ 错误
{
  instance: InstanceEnum.lightScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'  // ❌ lightScene 不需要
  },
  parameters: {
    dataType: 'ENUM',
    options: []
  }
}
```

**修正**: 删除 `payloadDefine`

---

### ❌ 错误 2: diyScene 使用了 STRUCT 结构

```javascript
// ❌ 错误
{
  instance: InstanceEnum.diyScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'  // ❌ 不需要
  },
  parameters: {
    dataType: 'STRUCT',  // ❌ 应该是 ENUM
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

**修正**: 使用简单的 ENUM 结构，删除 payloadDefine 和 fields

---

### ❌ 错误 3: snapshot 缺少 payloadDefine

```javascript
// ❌ 错误
{
  instance: InstanceEnum.snapshot.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  // ❌ 缺少 payloadDefine
  parameters: {
    dataType: 'ENUM',
    options: []
  }
}
```

**修正**: 添加 `payloadDefine: { cmdType: 'ptReal' }`

---

### ❌ 错误 4: 编造了 options 数据

```javascript
// ❌ 错误
{
  instance: InstanceEnum.lightScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  parameters: {
    dataType: 'ENUM',
    options: [  // ❌ 禁止编造数据
      { name: 'Rainbow', value: 1 },
      { name: 'Sunset', value: 2 }
    ]
  }
}
```

**修正**: options 必须为空数组 `[]`

---

## 记忆口诀

**lightScene 和 diyScene**:
- ENUM 类型
- options 空数组
- 不要 payloadDefine

**snapshot**:
- ENUM 类型
- options 空数组
- 需要 payloadDefine

**所有外部填充字段**:
- 不要编造数据
- 由外部系统管理

---

## 检查清单

配置外部填充字段时，检查：

- [ ] lightScene 使用 ENUM，options 为 `[]`，无 payloadDefine
- [ ] diyScene 使用 ENUM，options 为 `[]`，无 payloadDefine
- [ ] snapshot 使用 ENUM，options 为 `[]`，有 `payloadDefine: { cmdType: 'ptReal' }`
- [ ] 所有字段的 options 都是空数组，没有编造数据
- [ ] diyScene 没有使用 STRUCT 结构

---

## 参考文档

- 详细说明: `external-data-fields.md`
- 完整 Skill 文档: `SKILL.md`
- H1232 示例: `H1232-template.md`

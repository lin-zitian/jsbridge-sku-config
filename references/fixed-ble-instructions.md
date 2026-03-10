# 固定蓝牙指令能力列表

本文档列出所有使用固定蓝牙指令的 instance，这些 instance **不需要**用户提供 BLE 指令配置。

## 固定指令能力（无需 action 配置）

以下 instance 使用标准的 cmdType，不需要配置 action：

### 1. powerSwitch (电源开关)
- **cmdType**: `turn`
- **说明**: 标准开关控制
- **无需 action 配置**

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

### 2. brightness (亮度调节)
- **cmdType**: `brightness`
- **说明**: 标准亮度控制
- **无需 action 配置**

```javascript
{
  instance: InstanceEnum.brightness.value,
  type: 'devices.capabilities.range',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'brightness'
  },
  parameters: {
    dataType: 'INTEGER',
    unit: 'unit.percent',
    range: { min: 1, max: 100, precision: 1 }
  }
}
```

---

### 3. colorRgb (RGB 颜色)
- **cmdType**: `colorwc` 或 `color`
- **说明**: 标准 RGB 颜色控制
- **无需 action 配置**

```javascript
{
  instance: InstanceEnum.colorRgb.value,
  type: 'devices.capabilities.color_setting',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'colorwc'  // 或 'color'
  },
  parameters: {
    dataType: 'INTEGER',
    range: { min: 0, max: 0xffffff, precision: 1 }
  }
}
```

---

### 4. colorTemperatureK (色温调节)
- **cmdType**: `colorwc`
- **说明**: 标准色温控制
- **无需 action 配置**

```javascript
{
  instance: InstanceEnum.colorTemperatureK.value,
  type: 'devices.capabilities.color_setting',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'colorwc'
  },
  parameters: {
    dataType: 'INTEGER',
    range: { min: 2700, max: 6500, precision: 1 }
  }
}
```

---

### 5. lightScene (灯光场景)
- **cmdType**: `ptReal`
- **说明**: 标准场景控制，options 由外部填充
- **无需 action 配置**

```javascript
{
  instance: InstanceEnum.lightScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  parameters: {
    dataType: 'ENUM',
    options: []  // 外部填充
  }
}
```

---

### 6. diyScene (DIY 场景)
- **cmdType**: `ptReal`
- **说明**: 标准 DIY 场景控制，options 由外部填充
- **无需 action 配置**

```javascript
{
  instance: InstanceEnum.diyScene.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  parameters: {
    dataType: 'ENUM',
    options: []  // 外部填充
  }
}
```

---

### 7. snapshot (快照)
- **cmdType**: `ptReal`
- **说明**: 标准快照功能，options 由外部填充
- **需要 payloadDefine**

```javascript
{
  instance: InstanceEnum.snapshot.value,
  type: 'devices.capabilities.dynamic_scene',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'ENUM',
    options: []  // 外部填充
  }
}
```

---

## 需要 action 配置的能力

以下 instance **必须**配置 action（BLE 指令）：

### 灯光类

1. **segmentedColorRgb** (分段颜色)
   - 需要配置 bleDefine 和 valIndex
   - 需要询问段数

2. **segmentedBrightness** (分段亮度)
   - 需要配置 bleDefine 和 valIndex
   - 需要询问段数

3. **musicMode** (音乐模式)
   - **必须配置 action**
   - musicMode 字段 options 为空数组
   - 后续填充音乐模式数据

4. **gradientToggle** (渐变开关)
   - 需要配置 bleDefine 和 valIndex

5. **mainLightToggle** (主灯开关)
   - 需要配置 bleDefine 和 valIndex

6. **backgroundLightToggle** (背灯开关)
   - 需要配置 bleDefine 和 valIndex

7. **nightlightToggle** (夜灯开关)
   - 需要配置 bleDefine 和 valIndex

8. **leftLightToggle** (左灯柱)
   - 需要配置 bleDefine 和 valIndex

9. **rightLightToggle** (右灯柱)
   - 需要配置 bleDefine 和 valIndex

### 其他设备类

10. **workMode** (工作模式)
    - 部分设备需要配置 action
    - 需要根据设备类型确认

11. **humidity** (湿度调节)
    - 部分设备需要配置 action
    - 需要根据设备类型确认

12. **iceMakingToggle** (制冰开关)
    - 制冰机专用
    - 需要配置 action

13. **precoolToggle** (预冷开关)
    - 制冰机专用
    - 需要配置 action

14. **nightlightScene** (夜灯场景)
    - 需要配置 action
    - 场景选项需要用户提供

15. **其他自定义 toggle 类**
    - 所有自定义的开关类能力
    - 都需要配置 action

---

## 配置规则

### 规则 1: 固定指令能力
如果 instance 在"固定指令能力"列表中：
- ✅ 不需要配置 action
- ✅ 只需要配置 payloadDefine.cmdType
- ✅ 配置 parameters

### 规则 2: 需要 action 的能力
如果 instance 不在"固定指令能力"列表中：
- ⚠️ **必须配置 action**
- 如果用户暂时无法提供 BLE 指令：
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
- 在任务清单中标记需要用户提供

### 规则 3: 特殊情况
- **musicMode**: 即使 musicMode 字段 options 为空，也必须配置完整的 action
- **lightScene/diyScene/snapshot**: 虽然不需要 action，但 options 必须为空数组

---

## 判断流程

```
用户选择的 instance
    ↓
是否在固定指令列表中？
    ↓
  是 → 不需要 action，只配置 cmdType
    ↓
  否 → 需要 action
    ↓
用户是否提供了 BLE 指令？
    ↓
  是 → 配置完整的 action
    ↓
  否 → 配置空 action + TODO 注释
       在任务清单中标记
```

---

## 示例

### 示例 1: 固定指令（powerSwitch）
```javascript
{
  instance: InstanceEnum.powerSwitch.value,
  type: 'devices.capabilities.on_off',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'turn'  // ✅ 固定 cmdType，无需 action
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

### 示例 2: 需要 action（gradientToggle）
```javascript
{
  instance: InstanceEnum.gradientToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  action: {  // ⚠️ 必须配置
    write: [{ bleDefine: ['33', 'a3', '01'], valIndex: [2] }]
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

### 示例 3: 暂无 BLE 指令（自定义能力）
```javascript
{
  instance: 'customToggle',
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  action: {
    // TODO: 请提供 BLE 指令配置
    // write: [
    //   {
    //     bleDefine: ['33', '...'],
    //     valIndex: [2, 3]
    //   }
    // ]
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

---

## 任务清单模板

当配置包含需要 action 的能力时，在任务清单中添加：

```markdown
### ⚠️ 需要配置 BLE 指令的能力

以下 instance 需要提供 BLE 指令配置：

- [ ] **gradientToggle**: 当前为空 action + TODO 注释
  - 需要提供: bleDefine 数组和 valIndex 数组
  - 参考: H1232 的 gradientToggle 配置
  
- [ ] **mainLightToggle**: 当前为空 action + TODO 注释
  - 需要提供: bleDefine 数组和 valIndex 数组
  - 参考: H1232 的 mainLightToggle 配置

### 📝 下一步操作
1. 准备 BLE 指令数据
2. 更新对应 instance 的 action 配置
3. 删除 TODO 注释
4. 测试 BLE 指令是否正确
```

---

## 参考文档

- BLE 指令配置指南: `references/ble-instruction-guide.md`
- H1232 完整示例: `references/H1232-template.md`
- 外部填充数据说明: `references/external-data-fields.md`

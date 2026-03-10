# 灯光(Light)配置指南

## 设备类型
- **类型标识**: `devices.types.light`
- **适用设备**: 智能灯带、灯泡、灯条、氛围灯

## 参考 SKU
- **H1232**: 标准 RGBIC 灯带，支持分段控制、音乐模式、场景模式

## 常用 Instance 能力

### 1. 基础控制
- `powerSwitch`: 电源开关
- `brightness`: 亮度调节(1-100)
- `colorRgb`: RGB 颜色调节
- `colorTemperatureK`: 色温调节(K值)

### 2. 多灯头控制
- `mainLightToggle`: 主灯开关
- `backgroundLightToggle`: 背灯开关
- `nightlightToggle`: 夜灯开关
- `leftLightToggle`: 左灯柱开关
- `rightLightToggle`: 右灯柱开关
- `pillarLightToggle`: 灯柱开关
- `baseLightToggle`: 底座开关
- `light1Toggle/light2Toggle/light3Toggle`: 三灯头控制

### 3. 分段控制
- `segmentedColorRgb`: 分段颜色调节
- `segmentedBrightness`: 分段亮度调节

### 4. 场景模式
- `lightScene`: 灯效场景(预设场景)
- `diyScene`: 自定义灯效场景
- `snapshot`: 快照(保存当前状态)

### 5. 音乐模式
- `musicMode`: 音乐模式(律动/节奏/能量等)

### 6. 其他功能
- `gradientToggle`: 渐变开关
- `workMode`: 工作模式

## BLE 指令配置

### 一致性指令(所有灯光设备通用)

#### powerSwitch - 电源开关
```javascript
action: {
  write: [{ bleDefine: ['33', '01'], valIndex: [2] }],
  read: ['aa', '01']
},
payloadDefine: { cmdType: 'turn' }
```

#### brightness - 亮度调节
```javascript
action: {
  write: [{ bleDefine: ['33', '02'], valIndex: [2] }],
  read: ['aa', '02']
},
payloadDefine: { cmdType: 'brightness' }
```

#### colorRgb - RGB 颜色
```javascript
action: {
  write: [{ bleDefine: ['33', '05', '01', '00', '00', '00'], valIndex: [3, 4, 5] }],
  read: ['aa', '05', '01']
},
payloadDefine: { cmdType: 'color' }
```

#### colorTemperatureK - 色温调节
```javascript
action: {
  write: [{ bleDefine: ['33', '05', '02', '00', '00'], valIndex: [3, 4] }],
  read: ['aa', '05', '02']
},
payloadDefine: { cmdType: 'colorwc' }
```

#### lightScene - 灯效场景
```javascript
action: {
  write: [{ bleDefine: ['33', '05', '04'], valIndex: [3] }],
  read: ['aa', '05', '04']
},
payloadDefine: { cmdType: 'ptReal' },
parameters: {
  dataType: 'ENUM',
  options: []  // 外部系统填充，保持为空数组
}
```

**注意**: options 由外部系统填充，不要编造数据

#### diyScene - 自定义场景
```javascript
action: {
  write: [{ bleDefine: ['33', '05', '05', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00'], valIndex: [3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19] }],
  read: ['aa', '05', '05']
},
payloadDefine: { cmdType: 'ptReal' },
parameters: {
  dataType: 'ENUM',
  options: []  // 外部系统填充，保持为空数组
}
```

**注意**: options 由外部系统填充，不要编造数据

#### snapshot - 快照
```javascript
action: {
  write: [{ bleDefine: ['33', '05', '06'], valIndex: [3] }],
  read: ['aa', '05', '06']
},
payloadDefine: { cmdType: 'ptReal' },
parameters: {
  dataType: 'ENUM',
  options: []  // 外部系统填充，保持为空数组
}
```

**注意**: options 由外部系统填充，不要编造数据

### 需要 BLE 配置的指令

#### segmentedColorRgb - 分段颜色
```javascript
action: {
  write: [{ bleDefine: ['a3', '05', '0b', '00', '00', '00', '00', '00', '00', '00', '00', '00'], valIndex: [3, 4, 5, 6, 7, 8, 9, 10, 11] }],
  read: ['aa', '05', '0b']
},
payloadDefine: { cmdType: 'multiSync' }
```

**注意**: 需要询问用户段数(常见: 10段, 15段, 20段, 40段)

#### segmentedBrightness - 分段亮度
```javascript
action: {
  write: [{ bleDefine: ['a3', '05', '0c', '00', '00', '00', '00', '00', '00', '00'], valIndex: [3, 4, 5, 6, 7, 8, 9] }],
  read: ['aa', '05', '0c']
},
payloadDefine: { cmdType: 'multiSync' }
```

**注意**: 需要询问用户段数

#### musicMode - 音乐模式
```javascript
action: {
  write: [{ bleDefine: ['a3', '05', '0e', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00', '00'], valIndex: [3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19] }],
  read: ['aa', '05', '0e']
},
payloadDefine: { cmdType: 'multiSync' }
```

**注意**: 
- 音乐模式数据通常后期提供
- 初始配置时添加 TODO 标记
- 使用 `modeLanguageConst` 枚举类定义模式名称
- 参考 H1232 实现结构

#### gradientToggle - 渐变开关
```javascript
action: {
  write: [{ bleDefine: ['3a', '05', '0a', '00'], valIndex: [3] }],
  read: ['aa', '05', '0a']
},
payloadDefine: { cmdType: 'multiSync' }
```

**注意**: 需要 BLE 配置

### 多灯头控制指令

#### mainLightToggle - 主灯开关
```javascript
action: {
  write: [{ bleDefine: ['3a', '1b', '02', '01', '01'], valIndex: [4] }],
  read: ['aa', '1b', '02', '01']
},
payloadDefine: { cmdType: 'multiSync' }
```

**注意**: 不同灯头的指令地址不同,需要根据具体型号配置

## 配置流程

### Step 1: 确认基础信息
询问用户:
1. SKU 编号
2. goodsType(产品类型)
3. 灯光类型(灯带/灯泡/灯条等)

### Step 2: 确认颜色控制
询问用户:
1. 是否支持 RGB 颜色? (colorRgb)
2. 是否支持色温调节? (colorTemperatureK)
3. 如果支持色温,范围是多少? (默认2000-9000K)

### Step 3: 确认分段控制
询问用户:
1. 是否支持分段控制?
2. 如果支持,有多少段? (10/15/20/40段)
3. 是否支持分段颜色? (segmentedColorRgb)
4. 是否支持分段亮度? (segmentedBrightness)

### Step 4: 确认场景模式
询问用户:
1. 是否支持预设场景? (lightScene)
2. 是否支持自定义场景? (diyScene)
3. 是否支持快照功能? (snapshot)

### Step 5: 确认音乐模式
询问用户:
1. 是否支持音乐模式? (musicMode)
2. 如果支持,音乐模式数据是否已提供?
3. 如果未提供,musicMode 字段的 options 保持为空数组
4. **重要**: 即使 options 为空,也必须配置完整的 action (BLE 指令)

### Step 6: 确认多灯头控制
询问用户:
1. 是否有多个灯头?
2. 如果有,有哪些灯头? (主灯/背灯/夜灯/左右灯柱等)

### Step 7: 确认其他功能
询问用户:
1. 是否支持渐变功能? (gradientToggle)
2. 是否有其他特殊功能?

### Step 8: BLE 指令配置
- 基础指令(powerSwitch/brightness/colorRgb/colorTemperatureK/lightScene/diyScene/snapshot)使用标准配置
- 分段控制(segmentedColorRgb/segmentedBrightness)需要配置段数
- 音乐模式(musicMode)需要配置或添加 TODO
- 多灯头控制需要根据具体灯头配置指令地址
- gradientToggle 需要 BLE 配置

## 常见配置模式

### 模式 1: 基础灯光
- powerSwitch
- brightness
- colorRgb
- colorTemperatureK

### 模式 2: 场景灯光
- 基础灯光 +
- lightScene
- diyScene
- snapshot

### 模式 3: 分段灯光
- 场景灯光 +
- segmentedColorRgb
- segmentedBrightness
- gradientToggle

### 模式 4: 音乐灯光
- 分段灯光 +
- musicMode

### 模式 5: 多灯头灯光
- 基础灯光 +
- mainLightToggle
- backgroundLightToggle
- nightlightToggle
- 其他灯头开关

## 注意事项

1. **cmdType**: 
   - powerSwitch 使用 `turn`
   - brightness 使用 `brightness`
   - colorRgb 使用 `color`
   - colorTemperatureK 使用 `colorwc`
   - lightScene/diyScene/snapshot 使用 `ptReal`
   - 其他使用 `multiSync`

2. **外部填充数据**:
   - lightScene、diyScene、snapshot 的 options 必须为空数组 `[]`
   - musicMode 的 musicMode 字段 options 必须为空数组 `[]`
   - **禁止编造这些字段的数据**

3. **音乐模式**: 
   - musicMode 字段 options 为空数组
   - **必须配置完整的 action** (BLE 指令)
   - 使用 `modeLanguageConst` 枚举
   - 参考 H1232 结构

4. **色温范围**: 默认2000-9000K,需要确认具体范围

5. **分段数量**: 常见10/15/20/40段,需要确认

6. **多灯头**: 不同灯头指令地址不同,需要确认

7. **ctrlPlatformSupported**: 新 SKU 只使用 `['openApi']`

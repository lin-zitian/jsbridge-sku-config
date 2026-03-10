# SKU 配置工作流程

本文档详细说明 SKU 配置的完整工作流程和任务清单机制。

## 配置流程概览

```
用户提供 SKU 信息
    ↓
验证 Instance
    ↓
详细配置确认
    ↓
生成配置文件
    ↓
返回任务清单
    ↓
后续配置（如音乐模式）
```

---

## 步骤 1: 用户提供信息

用户需要提供：
- SKU 编号（如：H7200）
- 需要支持的功能实例列表
- 产品类别（如果是制冰机，需要特别注意）

示例：
```
SKU: H7200
功能: powerSwitch, brightness, colorRgb, colorTemperatureK, musicMode
```

**特殊设备类型**：
- 如果是制冰机（`devices.types.ice_maker`），参考 `ice-maker-guide.md`
- 制冰机的 BLE 指令差异较大，需要更详细的配置流程

---

## 步骤 2: 详细配置确认

### 2.1 基础信息
- ✅ 产品类别（默认：devices.types.light）
- ✅ 产品名称
- ✅ **goodsType**（必须询问）
  - 数据类型：数字（Number）
  - 用户提供时：使用数字（如：69, 321, 184）
  - 用户未提供时：使用空字符串 ''

### 2.2 色温配置（如果有 colorTemperatureK）
询问用户：
- 色温最小值（默认：2700K）
- 色温最大值（默认：6500K）

### 2.3 分段控制配置（如果有分段功能）
询问用户：
- 分段颜色控制支持多少段？（如：13段，segment: 0-12）
- 分段亮度控制支持多少段？

### 2.4 音乐模式配置（如果有 musicMode）
- 当前：添加 TODO 标记
- 配置 BLE 指令（bleDefine 和 valIndex）
- 后续：用户提供音乐模式数据后再配置

### 2.5 BLE 指令配置（如果有需要自定义指令的 instance）

**重要**：只有当用户选择的 instance 需要自定义 BLE 指令时，才询问用户配置。

详细指南：`references/ble-instruction-guide.md`

#### 需要配置 BLE 指令的 Instance

如果用户选择了以下 instance，必须配置 BLE 指令：
- `segmentedColorRgb` (分段颜色)
- `segmentedBrightness` (分段亮度)
- `musicMode` (音乐模式)
- `gradientToggle` (渐变开关)
- `mainLightToggle` (主灯开关)
- `backgroundLightToggle` (背灯开关)
- `leftLightToggle` / `rightLightToggle` (左右灯)
- `nightlightToggle` (夜灯开关)
- `workMode` (工作模式)
- `humidity` (湿度控制)

#### 询问用户

对于每个需要配置的 instance：

1. **BLE 指令** (bleDefine)
   ```
   请提供 [instanceName] 的 BLE 指令（十六进制字符串数组）
   示例：['33', '05', '15', '01', 'ff', 'ff', 'ff']
   ```

2. **数据位置索引** (valIndex)
   ```
   请提供 [instanceName] 的数据位置索引（数字数组）
   示例：[4, 5, 6]
   ```

3. **特殊参数**（根据 instance 类型）
   - 分段控制：段数
   - 工作模式：模式列表
   - 湿度控制：范围

#### 提供参考值

如果用户暂时无法提供：
- 使用参考 SKU（如 H1232）的指令
- 在任务清单中标记需要确认
- 提醒用户后续需要更新

---

## 步骤 3: 生成配置文件

生成以下文件：

### package.json
```json
{
  "main": "./dist/[SKU].global.js",
  "license": "MIT",
  "buildOptions": {
    "name": "[SKU]",
    "formats": ["global"]
  }
}
```

### README.md
包含所有 instance 的使用示例

### src/index.mjs
包含：
- Endpoint 配置（goodsType 为数字或空字符串）
  ```javascript
  const Endpoint = {
    productSku: 'H1234',
    productCategory: 'devices.types.light',
    productName: 'RGBIC Light',
    description: '',
    goodsType: 69,  // 数字类型，如果用户未提供则为 ''
    capabilities: [...]
  };
  ```
- MusicMode 常量（如果需要，带 TODO）
- 核心函数实现
- 导出（只使用新方法）

---

## 步骤 4: 返回任务清单

**必须返回的任务清单格式：**

```markdown
## 📋 SKU [编号] 配置任务清单

### ✅ 已完成任务
- [x] 读取 InstanceEnum 验证功能实例
- [x] 创建 SKU 包结构
- [x] 生成配置文件
- [x] 配置 Endpoint 基础信息
- [x] 配置 capabilities

### 🔧 需要用户确认的配置
- [ ] 色温范围：min=2700K, max=6500K
- [ ] 分段控制：13段（0-12）

### ⏳ 待后续配置任务
- [ ] 音乐模式数据（已添加 TODO）

### ⚠️ 新 Instance 需要添加
- [ ] 添加到 utils/common.js

### 📝 下一步操作建议
1. 检查生成的代码
2. 调整参数范围
3. 测试功能
```

---

## 任务清单详解

### ✅ 已完成任务
列出所有已经完成的配置步骤：
- 文件创建
- 基础配置
- Capabilities 配置

### 🔧 需要用户确认的配置
列出需要用户确认或可能需要调整的配置：
- 色温范围
- 分段数量
- 自定义 BLE 指令

### ⏳ 待后续配置任务
列出暂时无法完成，需要后续配置的任务：
- 音乐模式数据
- 其他延迟配置项

### ⚠️ 新 Instance 需要添加
如果创建了新的 instance，提醒用户添加到 InstanceEnum

### 📝 下一步操作建议
提供具体的后续操作指导

---

## 音乐模式配置流程

### 初次配置（添加 TODO）

1. **在 MusicMode 常量中添加 TODO**
```javascript
// TODO: 后续配置音乐模式时填充
const MusicMode = [];
```

2. **在 capability options 中添加 TODO**
```javascript
options: []  // TODO: 后续配置音乐模式时填充
```

3. **在任务清单中标记**
```markdown
### ⏳ 待后续配置任务
- [ ] 音乐模式数据待配置
  - 唤醒方式："配置 [SKU] 的音乐模式"
  - 需要提供：音乐模式名称和 BLE 指令
  - 参考：H1232 的 MusicMode 结构
```

### 后续配置（填充数据）

用户唤醒 skill：
```
"配置 H7200 的音乐模式"
```

用户提供数据：
```
音乐模式：
1. Ripple - ['base64string1', 'base64string2', 'base64string3']
2. Gridding - ['base64string1', 'base64string2', 'base64string3']
```

Skill 操作：
1. 读取现有的 SKU 配置文件
2. 填充 MusicMode 数组
3. 更新 musicCapabilitiesInterceptor
4. 更新 musicModeMqttPayloadInterceptor
5. 确保 getMusicMode 函数正确实现

---

## 配置检查清单

配置完成后，确保：

### 文件结构
- [ ] packages/[SKU]/package.json 存在
- [ ] packages/[SKU]/README.md 存在
- [ ] packages/[SKU]/src/index.mjs 存在

### 代码质量
- [ ] 只使用新方法（getMqttPayload、getCapabilities、getFullInstanceState）
- [ ] 不包含旧方法（getBleRead、getBleCommands）
- [ ] 所有 instance 都在 InstanceEnum 中定义
- [ ] ctrlPlatformSupported 只包含 ['openApi']

### 配置完整性
- [ ] Endpoint 信息完整（productSku、productCategory、productName、goodsType）
- [ ] goodsType 数据类型正确（数字或空字符串 ''）
- [ ] 所有 capabilities 配置正确
- [ ] 色温范围合理（如果有）
- [ ] 分段数量正确（如果有）
- [ ] 音乐模式 TODO 标记清晰（如果有）

### 文档完整性
- [ ] README.md 包含所有 instance 的使用示例
- [ ] 任务清单已返回给用户
- [ ] 后续配置步骤说明清楚

---

## 常见配置场景

### 场景 1: 基础灯具（无音乐模式）
功能：powerSwitch, brightness, colorRgb, colorTemperatureK

配置要点：
- 确认色温范围
- 无需音乐模式配置
- 任务清单简单

### 场景 2: RGBIC 灯带（有分段控制）
功能：powerSwitch, brightness, segmentedColorRgb, segmentedBrightness

配置要点：
- 确认分段数量
- 配置 segment 数组范围
- 配置 elementRange

### 场景 3: 音乐灯（有音乐模式）
功能：powerSwitch, brightness, colorRgb, musicMode

配置要点：
- 添加音乐模式 TODO
- 在任务清单中标记待配置
- 提供后续配置方式

### 场景 4: 全功能灯具
功能：powerSwitch, brightness, colorRgb, colorTemperatureK, segmentedColorRgb, musicMode

配置要点：
- 确认色温范围
- 确认分段数量
- 添加音乐模式 TODO
- 任务清单完整

---

## 错误处理

### Instance 不存在
- 询问用户 instance 详细信息
- 记录需要添加到 InstanceEnum
- 在任务清单中标记

### 配置冲突
- 提示用户冲突的配置
- 提供建议的解决方案
- 等待用户确认

### 缺少必要信息
- 明确列出缺少的信息
- 提供默认值建议
- 等待用户补充

---

## 最佳实践

1. **始终返回任务清单**
   - 每次配置后都返回
   - 格式统一
   - 信息完整

2. **明确标记 TODO**
   - 音乐模式使用 TODO 注释
   - 在任务清单中说明
   - 提供后续配置方式

3. **使用枚举类**
   - 音乐模式名称使用 modeLanguageConst
   - 保持一致性
   - 便于维护

4. **参考现有实现**
   - 音乐模式参考 H1232
   - 分段控制参考 H1232
   - 保持代码风格一致

5. **验证配置**
   - 检查 instance 是否存在
   - 验证参数范围合理性
   - 确保导出方法正确
